# acme-haproxy-clustered
Clustered version of the HAProxy deployment script for acme.sh.

We use this in the Riff.CC Lab to deploy and renew TLS certificates from Let's Encrypt without having to specifically account for them in the HAProxy configuration.

It's best used with clustered storage.

## What it does
* It's a post-deployment hook for acme.sh that copies your certificates to the remote hosts you specify.
* It then informs haproxy that it has new certificates using the admin socket.
* This allows you to load new certificates in real-time without so much as a reload!

## Usage
* Install acme.sh using whatever method you prefer. Install haproxy, preferably using [haproxy-cluster-playbook](https://github.com/riffcc/haproxy-cluster-playbook)
* Mount `/etc/haproxy/certs` on each of your HAProxy hosts using shared storage (MooseFS, CephFS, NFS will all work just fine).
* Copy `haproxy_clustered.sh` to `.acme.sh/deploy/haproxy_clustered.sh` and ensure it has an execute flag set (`chmod +x .acme.sh/deploy/haproxy_clustered.sh`)
* Ensure the user you're logged in as has key-based SSH access to the remote machines you want to update. We suggest creating a user called "haproxy_admin" on each of your HAProxy hosts, and adding it to the `haproxy` POSIX group. This will allow you to use the `haproxy` group to control access to the HAProxy stats socket.
* Run the deploy script, here's an example of how to use it
```
DEPLOY_HAPROXY_HOT_UPDATE=yes \
    DEPLOY_HAPROXY_STATS_SOCKET=/var/run/haproxy/admin.sock \
    DEPLOY_HAPROXY_PEM_PATH=/etc/haproxy/certs \
    DEPLOY_HAPROXY_REMOTE_MACHINES=haproxy_admin@haproxy01.riff.cc,haproxy_admin@haproxy02.riff.cc,haproxy_admin@haproxy03.riff.cc
    acme.sh --deploy -d riff.cc -d *.riff.cc --deploy-hook haproxy_clustered
```

## Examples
Here is an example of a successful run, deploying a certificate to three HAProxy hosts.
```
root@prod-haproxy01:~# DEPLOY_HAPROXY_HOT_UPDATE=yes DEPLOY_HAPROXY_STATS_SOCKET=/var/run/haproxy/admin.sock DEPLOY_HAPROXY_PEM_PATH=/etc/haproxy/certs   DEPLOY_HAPROXY_REMOTE_MACHINES=haproxy_admin@haproxy01.riff.cc,haproxy_admin@haproxy02.riff.cc,haproxy_admin@haproxy03.riff.cc acme.sh --deploy -d riff.cc -d *.riff.cc --deploy-hook haproxy_clustered
[Mon Oct  9 13:06:43 UTC 2023] The domain 'riff.cc' seems to have a ECC cert already, lets use ecc cert.
[Mon Oct  9 13:06:44 UTC 2023] Deploying PEM file
[Mon Oct  9 13:06:44 UTC 2023] Moving new certificate into place
Updating existing certificate '/etc/haproxy/certs/riff.cc.pem' over HAProxy stats socket.
Applying to remote machine: haproxy_admin@haproxy01.riff.cc
Updating existing certificate '/etc/haproxy/certs/riff.cc.pem' over HAProxy stats socket.
Applying to remote machine: haproxy_admin@haproxy02.riff.cc
Updating existing certificate '/etc/haproxy/certs/riff.cc.pem' over HAProxy stats socket.
Applying to remote machine: haproxy_admin@haproxy03.riff.cc
Updating existing certificate '/etc/haproxy/certs/riff.cc.pem' over HAProxy stats socket.
[Mon Oct  9 13:06:48 UTC 2023] Success
```