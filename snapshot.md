# ♻️ Snapshot

> **Info**  
> Updated every 6 hours.  
> Pruning: custom | 100/0/10 | indexer: null

Check snapshot height
```bash
echo "XRPL Snapshot Height: $(curl -s https://files.mictonode.com/xrpl-mainnet/snapshot/block-height.txt)"
```

```bash
sudo systemctl stop exrpd
cp $HOME/.exrpd/data/priv_validator_state.json $HOME/.exrpd/priv_validator_state.json.backup
rm -rf $HOME/.exrpd/data

SNAPSHOT_URL="https://files.mictonode.com/xrpl-mainnet/snapshot/"
LATEST_SNAPSHOT=$(curl -s $SNAPSHOT_URL | grep -oP 'xrpl-mainnet_\d+\.tar\.lz4' | sort -t_ -k2 -n | tail -n 1)

if [ -n "$LATEST_SNAPSHOT" ]; then
  FULL_URL="${SNAPSHOT_URL}${LATEST_SNAPSHOT}"
  if curl -s --head "$FULL_URL" | head -n 1 | grep "200" > /dev/null; then
    curl "$FULL_URL" | lz4 -dc - | tar -xf - -C $HOME/.exrpd
    
    mv $HOME/.exrpd/priv_validator_state.json.backup $HOME/.exrpd/data/priv_validator_state.json
    
    sudo systemctl restart exrpd && sudo journalctl -fu exrpd -o cat
  else
    echo "Snapshot URL is not accessible"
  fi
else
  echo "No snapshot found"
fi
```