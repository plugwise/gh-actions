# Plugwise Github Actions

## Workflow example 1 (not related to actions in this repo)

For python having environment variables set. This is the extended example for HA integrations, obviously retrieving the HA Core `pyproject.toml` and including it in the hashFiles is not needed for the python modules. Also see the note on actually using the actions here.

```test.yml
jobs:
  cache:
    runs-on: ubuntu-latest
    name: Cache identify
    outputs:
      cache-key: ${{ steps.set-key.outputs.cache-key }}
    steps:
      - name: Check out committed code
        uses: actions/checkout@v4
      - name: Set up Python ${{ env.DEFAULT_PYTHON }}
        id: python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.DEFAULT_PYTHON }}
      - name: Fetch HA pyproject
        id: core-version
        run: wget -O ha_pyproject.toml "https://raw.githubusercontent.com/home-assistant/core/refs/heads/dev/pyproject.toml"
      - name: Compute cache key
        id: set-key
        run: echo "cache-key=${{ runner.os }}-venv-cache-${{ env.CACHE_VERSION }}-${{ steps.python.outputs.python-version }}-${{ hashFiles('pyproject.toml', 'requirements_test.txt', '.pre-commit-config.yaml', 'ha_pyproject.toml') }}" >> "$GITHUB_OUTPUT"
```

## Workflow example 2 (caching for an integration)

```test.yml
  prepare:
    runs-on: ubuntu-latest
    needs: cache
    name: Prepare
    steps:
      - name: Prepare code checkout and python/pre-commit setup
        id: cache-reuse
        uses: plugwise/gh-actions/prepare-python-and-code@v1
        with:
          cache-key: ${{ needs.cache.outputs.cache-key }}
          fail-on-miss: false  # First time create cache (if not already exists)
          python-version: ${{ env.DEFAULT_PYTHON }}
          venv-dir: ${{ env.VENV }}
          precommit-home: ${{ env.PRE_COMMIT_HOME }}
          clone-core: "true"
```
