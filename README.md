# ey_tp

```bash
owner="eucalypsih";repo="ey_tp";git clone -q --filter=blob:none --no-checkout git@github.com:${owner}/${repo}.git && sleep 0.5 && cd $repo && sleep 0.5 && un="eucalypsih";ue="eucalypsih@gmail.com";git config user.name "$un" && sleep 0.5 && git config user.email "$ue" && sleep 0.5 && git config gpg.format ssh && sleep 0.5 && git config user.signingkey ~/.ssh/id_rsa.pub && sleep 0.5 && git config commit.gpgsign true && sleep 0.5 && git config gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers

```
`owner="eucalypsih";repo="ey_tp";git clone -q --filter=blob:none --no-checkout git@github.com:${owner}/${repo}.git && sleep 0.5 && cd $repo && sleep 0.5 && un="eucalypsih";ue="eucalypsih@gmail.com";git config user.name "$un" && sleep 0.5 && git config user.email "$ue" && sleep 0.5 && git config gpg.format ssh && sleep 0.5 && git config user.signingkey ~/.ssh/id_rsa.pub && sleep 0.5 && git config commit.gpgsign true && sleep 0.5 && git config gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers`

<br>

---

<br>



- `'termux-packages/packages/gawk/'`
```bash
git sparse-checkout set --no-cone 'termux-packages/packages/gawk/' && sleep 0.5 && git checkout main
```























<br>









