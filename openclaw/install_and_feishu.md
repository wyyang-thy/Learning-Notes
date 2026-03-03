# 记录下载openclaw以及接入飞书的步骤
## first
### install the openclaw with interaction. this step need very long time, about an hour.
```
curl -fsSL https://openclaw.ai/install.sh | bash
```
### there are some options need to be chosen and i chose 'yes' for all of them.

## second
### install feishu plugin
```
openclaw plugins install @openclaw/feishu
```
### restart the openclaw to load plugins
```
openclaw gateway restart
```
### checkout local
```
openclaw plugins install ./extensions/feishu
```
### install feishu
```
openclaw onboard
```
