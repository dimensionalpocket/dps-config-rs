# 25-remove-api-subdomain-config

## Overview

Remove the `api_subdomain` configuration property, its getter/setter methods, and related tests. The architecture has changed and this "api subdomain" is no longer required. Additionally, add a new `api_path` configuration property as a separate, unrelated feature for API path configuration.

## Files to Modify

### `src/lib.rs`

**Struct field changes:**
- Remove `api_subdomain: Option<String>,` from `DpsConfig` struct (line 44)
- Add `api_path: Option<String>,` to `DpsConfig` struct

**Constructor changes:**
- Remove `api_subdomain: load_env_string("DPS_API_SUBDOMAIN"),` from `DpsConfig::new()` (line 77)
- Add `api_path: load_env_string("DPS_API_PATH"),` to `DpsConfig::new()`

**Method removals:**
- Remove `get_api_subdomain()` method (lines 112-117)
- Remove `set_api_subdomain()` method (lines 119-122)
- Remove `get_api_domain()` method (lines 265-270)

**Method additions:**
- Add `get_api_path()` method returning default `"api"`
- Add `set_api_path()` method

**Method updates:**
- Update `get_auth_api_url()` to construct URLs as `{protocol}://{auth_api_subdomain}.{domain}/{api_path}`

**Documentation updates:**
- Remove `api_subdomain` from environment variables list in `DpsConfig::new()` doc comment (line 64)
- Add `DPS_API_PATH` to environment variables list in `DpsConfig::new()` doc comment
- Remove example usage in top-level documentation (line 27)

**Test removals:**
- Remove `assert_eq!(config.get_api_subdomain(), "api");` from `test_default_values()` (line 332)
- Remove `config.set_api_subdomain("api");` calls from:
  - `test_auth_api_url_without_port()` (line 377)
  - `test_auth_api_url_with_port()` (line 388)
- Remove entire `test_api_domain_computed()` test (lines 365-370) since `get_api_domain()` will be removed

**Test additions:**
- Add `assert_eq!(config.get_api_path(), "api");` to `test_default_values()`
- Add test for `set_api_path()` in `test_setters()`
- Add tests for `DPS_API_PATH` environment variable loading

## Methods Impacted by This Removal

The following methods depend on `api_subdomain` and will need attention:

1. **`get_api_domain()`** (lines 265-270)
   - Currently returns `{api_subdomain}.{domain}`
   - **This method should be entirely removed**

2. **`get_auth_api_url()`** (lines 272-286)
   - Currently uses `get_api_domain()` which depends on `api_subdomain`
   - Constructs URLs like `{protocol}://{auth_api_subdomain}.{api_subdomain}.{domain}`
   - **Should be updated to construct URLs as `{protocol}://{auth_api_subdomain}.{domain}`**
   - Will no longer depend on `get_api_domain()`

## Test Updates Required

**Tests that will fail after removal and need modification:**

1. **`test_auth_api_url_without_port()`** (lines 373-380)
   - Currently expects `https://auth.api.dps.localhost`
   - Should be updated to expect `https://auth.dps.localhost/api`

2. **`test_auth_api_url_with_port()`** (lines 383-394)
   - Currently expects `http://auth.api.dps.localhost:3000`
   - Should be updated to expect `http://auth.dps.localhost:3000/api`

3. **`test_readme_example()`** (lines 397-403)
   - Currently expects `http://auth.api.test.local:8080`
   - Should be updated to expect `http://auth.test.local:8080/api`

## README Updates Required

**Configuration table updates:**
- Remove `api_subdomain` row from configuration table
- Add new `api_path` row with:
  - Property: `api_path`
  - Environment Variable: `DPS_API_PATH`
  - Default: `"api"`
  - Description: Path (without leading slash) for API endpoints

**Method documentation updates:**
- Remove documentation for `get_api_subdomain()` / `set_api_subdomain()`
- Remove documentation for `get_api_domain()`
- Add documentation for `get_api_path()` / `set_api_path()`
- Update `get_auth_api_url()` documentation to reflect new URL structure

**Example code updates:**
- Remove any examples using `set_api_subdomain()`
- Update examples to show new URL structure with path
- Add examples using `set_api_path()` if appropriate

## Implementation Notes

- The `auth_api_subdomain` property remains intact
- New `api_path` property is a separate, unrelated feature - it does NOT replace `api_subdomain` functionality
- Environment variable `DPS_API_SUBDOMAIN` loading will be removed
- Environment variable `DPS_API_PATH` loading will be added
- Default value logic for `api_subdomain` will be removed
- Default value `"api"` will be used for `api_path`
- `get_api_domain()` method will be completely removed
- `get_auth_api_url()` will construct URLs as `{protocol}://{auth_api_subdomain}.{domain}/{api_path}`
- All code changes are contained within `src/lib.rs`
- README.md will need updates to reflect the new configuration structure
- No other files in the codebase reference `api_subdomain` directly