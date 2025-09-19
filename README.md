# workflows-canary-github-actions.ymlname: Digital Crystal Canary

# Runs every 15 minutes by default; adjust schedule if needed.
on:
  schedule:
    - cron: '*/15 * * * *'   # every 15 minutes
  workflow_dispatch: {}

permissions:
  contents: read
  actions: read

jobs:
  canary:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt || pip install web3

      - name: Run Digital Crystal enforcement (canary)
        env:
          # Provide these as encrypted repository secrets (Settings → Secrets)
          ETH_RPC_URL: ${{ secrets.ETH_RPC_URL }}
          SIGNER_ADDRESS: ${{ secrets.SIGNER_ADDRESS }}
          ANCHOR_PRIVATE_KEY: ${{ secrets.ANCHOR_PRIVATE_KEY }}  # optional; prefer HSM
          BROADCAST_ANCHOR: "false"   # keep false for canary mode
          ALLOW_THRESHOLD: "0.2"
          ESCALATE_THRESHOLD: "0.6"
        run: |
          python src/enforcement_engine.py .
