# Installing Nylas sync-engine in virtualenv

Tested by VCCloud

## Requirements

- Ubuntu 16.04
- Specification: 4c 4g
- Root privilege

## Installing dependency packages

```bash
apt-get update
apt-get -qq -y install --no-install-recommends         \
                       python-software-properties      \
                       python-dev                      \
                       python-pip                      \
                       build-essential                 \
                       libmysqlclient-dev              \
                       gcc                             \
                       g++                             \
                       libxml2-dev                     \
                       libxslt-dev                     \
                       lib32z1-dev                     \
                       libffi-dev                      \
                       pkg-config                      \
                       python-lxml                     \
                       liblua5.2-dev                   \
                       lua5.2                          \
                       python-setuptools               \
                       curl tmux git mysql-client      \
                       redis-server
```

## Setup

### Making data folders

```
mkdir -p /etc/inboxapp /var/lib/inboxapp
chown -R inbox:inbox /etc/inboxapp /var/lib/inboxapp
```

### Cloning codebase

```
cd /opt
git clone https://github.com/VCCloud/sync-engine.git inboxapp
chown -R inbox: /opt/inboxapp
```

### Setup virtualenv

Run as `inbox` user

```
cd /opt/inboxapp
su inbox
virtualenv .venv
echo "export PYTHONPATH=\".:\"" >> .venv/bin/activate
source .venv/bin/activate
pip install -r requirements.txt
pip install -U pyasn1
python setup.py install
```

### Configuration

Run as `inbox` user

```
cd /opt/inboxapp
su inbox
cp deploy/config.json.sample /etc/inboxapp/config.json
cp deploy/secrets.yml.sample /etc/inboxapp/secrets.yml
```

Mandatory keys are:

- `/etc/inboxapp/config.json`:
    + DATABASE_HOSTS

- `/etc/inboxapp/secrets.yml`:
    + DATABASE_USERS

### Init database

Run as `inbox` user

```
cd /opt/inboxapp
su inbox
source .venv/bin/activate
./bin/create-db
./bin/migrate-db
```

### Setup supervisor

Run as `root` user

```
apt-get install -y supervisor

cd /opt/inboxapp
mkdir /var/log/inboxapp
cp scripts/inboxapp.supervisor.conf /etc/supervisor/conf.d/inboxapp.conf
supervisorctl reread
supervisorctl update
```

Verify-ing: Please check `/var/log/inboxapp/inbox-sync.stdout.log`
`/var/log/inboxapp/inbox-api.stdout.log`
