## Secure Docker daemon with TLS

The Docker daemon can communicate with remote clients over a TLS(HTTPS) socket.
This is done by [providing
Docker](https://docs.docker.com/engine/security/protect-access/) with a trusted
CA certificate. The daemon will then only accept connections from client
authenticated by a certificate signed by the CA.

This script generates the CA certificate and server key and certificate, and
client key and certificate. This is useful for Docker engine management services
like
[Portainer](https://lemariva.com/blog/2019/12/portainer-managing-docker-engine-remotely)

### Usage
0. Download the script

```bash
$ git clone https://github.com/kencx/docker-tls-certs.git && cd docker-tls-certs
$ chmod u+x generate-certs
```

1. Input the IP address and DNS hostname of the host of the Docker daemon.

```bash
IP=192.168.86.100
HOST=domain.tld
```

2. Run the script `./generate-certs`. It generates all keys and certificates to
   `$CERT_DIR`.
3. Copy `ca.pem, server-cert.pem, server-key.pem` to `/etc/docker/certs.d/`.
4. Make them known to the Docker daemon using the `/etc/docker/daemon.json` file.

```json
{
  "tlsverify": true,
  "tlscacert": "/etc/docker/certs.d/ca.pem",
  "tlscert": "/etc/docker/certs.d/server-cert.pem",
  "tlskey": "/etc/docker/certs.d/server-key.pem",
  "hosts": ["tcp://0.0.0.0:2376", "unix:///var/run/docker.sock"]
}
```

5. Check the `/lib/systemd/system/docker.service` file

```bash
# docker.service
[Service]
ExecStart=/usr/bin/dockerd
...
```

6. Restart the daemon

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker.service
$ sudo systemctl status docker.service
```

7. Run a test to ensure the connection is working properly

```bash
$ docker -H $IP:2376 \
	--tlsverify \
	--tlscacert ~/.certs/ca.pem \
	--tlscert ~/.certs/cert.pem \
	--tlskey ~/.certs/key.pem \
	ps -a"
```

Here, the certs and keys are stored in `~/.certs`.

## References
All credits go to [Auto generation of docker service TLS
certificates](https://developpaper.com/automatic-generation-of-docker-service-tls-certificate/)
