# Phase 13: Hash-Based Store Keys

## Overview

Replace path-based asset keys with hash-based keys. The current implementation uses `PathSafeStr` constraints to build human-readable paths from job parameters. This change:

1. Replaces `PathSafeStr` with support for `str` and `frozenset[str]` param types
2. Uses SHA-256 hash of canonically serialized params instead of path segments
3. Adds `params.json` file for debugging opaque hashes

**Spec Reference:** [hash_store_keys.md](../hash_store_keys.md)

---

## Step 1: Update `JobParams` Validator

**File:** `src/kiln_pipelines/types.py`

### 1.1 Remove `PathSafeStr`

Remove the `PathSafeStr` type alias:

```python
# DELETE THIS:
# PathSafeStr = Annotated[str, Field(pattern=r"^[a-zA-Z0-9_-]+$")]
```

### 1.2 Rewrite `JobParams` Validator

Replace `_validate_all_fields_path_safe` with `_validate_param_types`:

```python
class JobParams(BaseModel, frozen=True):
    """
    Base class for job parameters.
    
    All fields MUST be typed as either:
    - str: Any UTF-8 string
    - frozenset[str]: Unordered set of strings (order-invariant for hashing)
    
    Example:
        class MyParams(JobParams):
            doc_id: str
            input_ids: frozenset[str]
    """
    
    model_config = ConfigDict(extra="forbid")
    
    @model_validator(mode="after")
    def _validate_param_types(self) -> Self:
        """
        Validate all fields are allowed types (str or frozenset[str]).
        
        This is a runtime safety net that catches type errors that the
        static type checker might miss.
        """
        for field_name in self.__class__.model_fields:
            value = getattr(self, field_name)
            
            if isinstance(value, str):
                continue  # String is allowed
                
            if isinstance(value, frozenset):
                # Validate all items in frozenset are strings
                for item in value:
                    if not isinstance(item, str):
                        raise TypeError(
                            f"JobParams field '{field_name}' is frozenset but contains "
                            f"non-string item of type {type(item).__name__}. "
                            f"Only frozenset[str] is allowed."
                        )
                continue
            
            # Not an allowed type
            raise TypeError(
                f"JobParams field '{field_name}' has type {type(value).__name__}. "
                f"Only str and frozenset[str] are allowed."
            )
        
        return self
```

### 1.3 Update `AssetKey`

Rename `params_path` to `params_hash`:

```python
class AssetKey(BaseModel, frozen=True):
    """Immutable identifier for an asset location."""
    
    tenant_id: str
    job_name: str
    job_version: str
    params_hash: str  # SHA-256 hash (64 hex chars) of canonically serialized params
    
    def _asset_suffix(self) -> str:
        """
        Asset-specific path suffix without root or tenant.
        Returns: job_<name>/v<version>/<params_hash>/
        """
        return f"job_{self.job_name}/v{self.job_version}/{self.params_hash}/"
    
    def storage_path(self, root: str) -> str:
        """Full path: root/tenant_<id>/job_<name>/v<version>/<params_hash>/"""
        return f"{root}/tenant_{self.tenant_id}/{self._asset_suffix()}"
```

---

## Step 2: Implement `params_to_hash`

**File:** `src/kiln_pipelines/job.py`

### 2.1 Add Hash Function

Add a module-level function for canonical serialization and hashing:

```python
import hashlib

def _canonical_serialize_to_hash(params: JobParams) -> str:
    """
    Generate deterministic SHA-256 hash from job parameters.
    
    Uses length-prefixed binary format:
    - Fields sorted alphabetically by name
    - Strings: type tag 's' + length-prefixed UTF-8 bytes
    - Sets: type tag 'S' + count + sorted length-prefixed items
    
    Returns:
        64-character lowercase hex string (SHA-256 hash)
    """
    h = hashlib.sha256()
    fields = sorted(params.__class__.model_fields.keys())
    
    # Field count
    h.update(len(fields).to_bytes(4, 'big'))
    
    for name in fields:
        value = getattr(params, name)
        
        # Field name (length-prefixed)
        name_bytes = name.encode('utf-8')
        h.update(len(name_bytes).to_bytes(4, 'big'))
        h.update(name_bytes)
        
        if isinstance(value, str):
            # String: type tag + length-prefixed value
            h.update(b's')
            val_bytes = value.encode('utf-8')
            h.update(len(val_bytes).to_bytes(4, 'big'))
            h.update(val_bytes)
            
        elif isinstance(value, frozenset):
            # Set: type tag + count + sorted length-prefixed items
            h.update(b'S')
            items = sorted(value)  # Sort for determinism
            h.update(len(items).to_bytes(4, 'big'))
            for item in items:
                item_bytes = item.encode('utf-8')
                h.update(len(item_bytes).to_bytes(4, 'big'))
                h.update(item_bytes)
        else:
            # Should never reach here if JobParams validation is correct
            raise TypeError(f"Unsupported param type for '{name}': {type(value)}")
    
    return h.hexdigest()
```

### 2.2 Replace `params_to_path` with `params_to_hash`

```python
class Job(ABC, Generic[P, I, O]):
    # ... existing code ...
    
    def params_to_hash(self, params: P) -> str:
        """
        Convert params to deterministic SHA-256 hash.
        
        Uses canonical binary serialization (length-prefixed, sorted fields/items)
        to ensure the same parameters always produce the same hash.
        
        Args:
            params: The job parameters to hash
            
        Returns:
            64-character lowercase hex string (SHA-256 hash)
        """
        return _canonical_serialize_to_hash(params)
```

### 2.3 Update Docstrings

Update the Job class docstring to remove references to `PathSafeStr`:

```python
class Job(ABC, Generic[P, I, O]):
    """
    Base class for all jobs. Subclass and define name, version, and run().

    Type parameters:
    - P: Params type (must extend JobParams, fields must be str or frozenset[str])
    - I: Inputs type (typed container for input AssetRefs, or None if no inputs)
    - O: Output type (must extend JobOutput)
    
    # ... rest of docstring ...
    """
```

---

## Step 3: Add `params.json` Writing

**File:** `src/kiln_pipelines/pipeline.py`

### 3.1 Add Serialization Helper

```python
import json

def _serialize_params_for_debug(params: JobParams) -> str:
    """
    Serialize params to JSON for debugging purposes.
    
    Note: This is NOT used for hashing - only for human inspection.
    frozenset values are converted to sorted lists for JSON compatibility.
    """
    data = {}
    for field_name in sorted(params.__class__.model_fields.keys()):
        value = getattr(params, field_name)
        if isinstance(value, frozenset):
            # Convert to sorted list for JSON
            data[field_name] = sorted(value)
        else:
            data[field_name] = value
    return json.dumps(data, indent=2, sort_keys=True)
```

### 3.2 Update `_save_to_storage`

Write `params.json` alongside `output.json`:

```python
async def _save_to_storage(
    self,
    key: AssetKey,
    output: JobOutput,
    params: JobParams,  # NEW parameter
    additional_files: dict[str, Path],
    attempt_uuid: str,
) -> None:
    # ... existing output.json writing ...
    
    # Write params.json for debugging
    params_json = _serialize_params_for_debug(params)
    await self.store.write_file(key, "params.json", params_json.encode("utf-8"), attempt_uuid)
    
    # ... rest of method unchanged ...
```

### 3.3 Update Callers

Update `_compute_and_commit` to pass `params` to `_save_to_storage`:

```python
save_task = asyncio.create_task(
    self._save_to_storage(key, output, params, stable_additional_files, attempt_uuid)
)
```

---

## Step 4: Update Key Construction

**File:** `src/kiln_pipelines/pipeline.py`

Update `materialize()` to use `params_to_hash` instead of `params_to_path`:

```python
async def materialize(self, job, params, inputs, ...):
    # Build asset key
    params_hash = job.params_to_hash(params)
    key = AssetKey(
        tenant_id=self.tenant_id,
        job_name=job.name,
        job_version=job.version,
        params_hash=params_hash,  # Changed from params_path
    )
    # ... rest unchanged ...
```

---

## Step 5: Update `__init__.py` Exports

**File:** `src/kiln_pipelines/__init__.py`

Remove `PathSafeStr` from exports:

```python
# Remove from __all__:
# "PathSafeStr",

# Remove from imports:
# from kiln_pipelines.types import PathSafeStr
```

Keep `JobParams` exported as it's still needed.

---

## Step 6: Update Example Files

**File:** `specs/examples/job_definitions.md`

Already updated in spec phase - verify examples use `str` and `frozenset[str]` instead of `PathSafeStr`.

**File:** `specs/examples/fanout_fanin_patterns.md`

Update any `PathSafeStr` references to `str`.

**File:** `specs/examples/pipeline_usage.md`

Update any `PathSafeStr` references to `str`.

---

## Tests

### Test File: `src/kiln_pipelines/test_types.py`

#### JobParams Validator Tests

- `test_job_params_accepts_string_field` — String field is valid
- `test_job_params_accepts_frozenset_str_field` — `frozenset[str]` field is valid
- `test_job_params_accepts_mixed_fields` — Both str and frozenset[str] in same params
- `test_job_params_rejects_int_field` — Integer field raises TypeError
- `test_job_params_rejects_list_field` — List field raises TypeError (must use frozenset)
- `test_job_params_rejects_set_field` — Mutable set raises TypeError (must use frozenset)
- `test_job_params_rejects_dict_field` — Dict field raises TypeError
- `test_job_params_rejects_frozenset_int` — `frozenset[int]` raises TypeError
- `test_job_params_rejects_frozenset_mixed` — frozenset with non-string item raises TypeError
- `test_job_params_accepts_empty_frozenset` — Empty frozenset is valid
- `test_job_params_accepts_unicode_strings` — Non-ASCII strings are valid
- `test_job_params_accepts_special_characters` — Strings with special chars are valid

#### AssetKey Tests

- `test_asset_key_uses_params_hash` — Field is named `params_hash`
- `test_asset_key_storage_path_includes_hash` — `storage_path()` includes hash in path

### Test File: `src/kiln_pipelines/test_job.py`

#### `params_to_hash` Tests

- `test_params_to_hash_deterministic` — Same params → same hash (multiple calls)
- `test_params_to_hash_field_order_independence` — Hash same regardless of class field order
- `test_params_to_hash_set_order_independence` — `frozenset({"a", "b"})` == `frozenset({"b", "a"})`
- `test_params_to_hash_type_distinction` — `"foo"` (str) and `frozenset({"foo"})` produce different hashes
- `test_params_to_hash_unicode` — Non-ASCII strings hash correctly
- `test_params_to_hash_empty_params` — Empty JobParams subclass produces valid hash
- `test_params_to_hash_empty_frozenset` — `frozenset()` handled correctly
- `test_params_to_hash_large_values` — Long strings and large sets work
- `test_params_to_hash_returns_64_hex_chars` — Result is 64 lowercase hex characters

#### Hash Stability Test (Regression)

- `test_params_to_hash_stability` — Hardcoded params → hardcoded expected hash
  ```python
  def test_params_to_hash_stability():
      """
      Regression test: ensure hash algorithm hasn't changed.
      If this test fails, existing stored assets will be orphaned.
      """
      class StableParams(JobParams):
          doc_id: str
          tags: frozenset[str]
      
      params = StableParams(doc_id="test123", tags=frozenset(["alpha", "beta"]))
      # This hash was computed once and hardcoded
      expected = "..."  # Compute actual value during implementation
      assert job.params_to_hash(params) == expected
  ```

### Test File: `src/kiln_pipelines/test_pipeline.py`

#### params.json Tests

- `test_params_json_written_on_commit` — File exists in asset folder after commit
- `test_params_json_contains_params` — File contains correct parameter values
- `test_params_json_frozenset_as_sorted_list` — frozenset serialized as sorted JSON array
- `test_params_json_not_used_for_hash` — Modifying params.json doesn't affect key lookup

#### Integration Tests

- `test_materialize_uses_hash_key` — Asset path contains hash, not path segments
- `test_materialize_same_params_same_key` — Identical params find same asset
- `test_materialize_different_params_different_key` — Different params create different assets

---

## Completion Criteria

- [ ] `PathSafeStr` removed from `types.py`
- [ ] `JobParams._validate_param_types` validates `str` and `frozenset[str]` only
- [ ] `JobParams._validate_param_types` validates frozenset items are strings
- [ ] `AssetKey.params_path` renamed to `params_hash`
- [ ] `AssetKey.storage_path()` uses hash in path
- [ ] `Job.params_to_path()` renamed to `Job.params_to_hash()`
- [ ] `_canonical_serialize_to_hash()` implements length-prefixed binary format
- [ ] `_serialize_params_for_debug()` added for params.json
- [ ] `_save_to_storage()` writes params.json
- [ ] `materialize()` uses `params_to_hash()` for key construction
- [ ] `PathSafeStr` removed from `__init__.py` exports
- [ ] `specs/examples/job_definitions.md` uses new types
- [ ] `specs/examples/fanout_fanin_patterns.md` updated
- [ ] `specs/examples/pipeline_usage.md` updated
- [ ] Hash stability regression test added
- [ ] All new tests pass
- [ ] Existing tests updated to use new types
- [ ] Type checking passes (`uvx ty check`)
- [ ] Lint/format passes (`uvx ruff check`, `uvx ruff format`)
