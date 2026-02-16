# Explanation

## What was the bug?

The `request()` method in `http_client.py` failed to refresh or use `dict`-typed OAuth2 tokens. When `oauth2_token` was a `dict`, API requests were sent without an `Authorization` header.

## Why did it happen?

The refresh condition checked two cases but missed `dict`:

- `not self.oauth2_token` → `False` for a non-empty dict (dicts are truthy in Python)
- `isinstance(self.oauth2_token, OAuth2Token)` → `False` for a dict

Neither matched, so `refresh_oauth2()` was never called. The header-setting code also required an `OAuth2Token` instance, so no `Authorization` header was set either.

## Why does the fix solve it?

Adding `or isinstance(self.oauth2_token, dict)` to the existing refresh condition explicitly handles the missing `dict` case while preserving the original logic for `None` and expired `OAuth2Token`. After the refresh, `oauth2_token` becomes a valid `OAuth2Token`, so the header is set correctly.

## One edge case tests don't cover

If `refresh_oauth2()` itself fails (e.g. a network error in a real implementation), `oauth2_token` would remain invalid and the request would still go out unauthenticated.
