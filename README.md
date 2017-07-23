# Working with remote docker daemon via a SSH tunel

Problem:
* you don't have docker installed locally but have installed a client, that can talk to docker (e.g. your IDE)
* you work on a thin client (e.g. your laptop) and want to access docker on a remote machine where you have 10x available resources

Solutions:
* enable docker API (anybody can access your docker daemon and do pretty much what the root user can do)
* enable docker API with TLS and a cert (research how it's done for your docker & distro version)
* bind docker daemon's socket locally via SSH

Here we present the 3rd solution. This way the traffic is encrypted and we don't have to do any acrobatics around certs:

1. On your **server where docker daemon is running** edit the file `/lib/systemd/system/docker.service`:

`$ sudo nano /lib/systemd/system/docker.service`

2. Look for the line that starts with `ExecStart` which might look like:

`ExecStart=/usr/bin/dockerd -H fd://`

We'll add a TCP socket so we can bind it locally on our client via SSH:
`-H tcp://127.0.0.1:{choose_your_port_number}`. The resulting line should look something like:

`ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:{port_number_you've_decided_to_use}`

3. Reload your settings:

`$ sudo systemctl daemon-reload && sudo service docker restart`

4. Bind the port **from your client**:

`$ ssh pinky -nNT -L 19851:127.0.0.1:{port_number_you've_decided_to_use} &`

Your should now be able to communicate with your remote docker via the TCP socket
(`http://127.0.0.1:{port_number_you've_decided_to_use}`). 
