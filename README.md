# DargStack Template
A template for Docker stack project layouts.

## Table of Contents
1. **[Project](#project)**
1. **[Stack](#stack)**
    1. **[Development](#development)**
    1. **[Production](#production)**

<a name="project"></a>

## Project
The main project folder should at least contain a `Dockerfile`.

<a name="stack"></a>

## Stack

<a name="stack-development"></a>

### Development

#### Certificates
HTTPS/SSL encryption requires certificates. Those can easily be generated using the [new-certificates.sh](https://gist.github.com/Dargmuesli/538a2c382c009f4620803679c8172c9d) script. The root certificate needs to be imported in your browser.

#### DNS
The default configuration assumes that the local deployment presents itself on the domain and subdomains of `project.test`. Therefore one needs to configure the local DNS resolution to make this address resolvable. This can either be done by simply adding this domain and all subdomains to the operation system's hosts file or by settings up a local DNS server. An advantage of the latter method is that subdomain wildcards can be used and thus not every subdomain needs to be defined separately.

Here is an example configuration for [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq) on [Arch Linux](https://www.archlinux.org/) that uses the local DNS server on top of the router's advertised DNS server:

`/etc/dnsmasq.conf`
```conf
# Use NetworkManager's resolv.conf
resolv-file=/run/NetworkManager/resolv.conf

# Limit to machine-wide requests
listen-address=127.0.0.1

# Wildcard DNS
address=/.test/127.0.0.1

# Enable logging (systemctl status dnsmasq)
#log-queries
```

`/etc/NetworkManager/NetworkManager.conf`
```conf
[main]

# Don't touch /etc/resolv.conf
rc-manager=unmanaged
```

#### Docker Secrets
Confidential data, like usernames and passwords, need to be accessible as [Docker secrets](https://docs.docker.com/engine/swarm/secrets/) to keep them out of the source code. These files, which contain the passwords' values, need to exist inside the `project-name_stack/development/secrets/` directory.

#### Stack
Simply [deploy the development stack](https://docs.docker.com/engine/reference/commandline/stack_deploy/) using the following command:

```bash
docker stack deploy -c [project-name]-stack/development/stack.yml [project-name]
```

<a name="stack-production"></a>

### Production

#### Docker Secrets
Don't use password files for production. Use the `docker secret create` command instead. PowerShell on Windows may add a carriage return at the end of strings piped to the command. A workaround can be that you create secrets from temporary files that do not contain a trailing newline. They can be written using:

```PowerShell
"secret data" | Out-File secret_name -NoNewline
```

When done, shred those files!

#### Environment Variables
You may need to clone a `project-name_stack/production/.env.template` file to a `.env` file next to it and specify the included environment variables.

`.env` contains environment variables for the stack file itself. The `deploy.sh` script, mentioned in the **Production/Stack** section, executes a command similar to this for deployment where `-E` indicates preserved environment variables for `sudo` use:

```Bash
export $(cat .env | xargs) && sudo -E docker stack deploy -c stack.yml [project_name]
```

`traefik.env` sets provider credentials for DNS authentication as environment variables for the traefik service.

#### Stack
Utilize [new-production-stack.sh](https://gist.github.com/Dargmuesli/517d2032c1b148cf7a85e0cdccaa1818) to derive `project-name_stack/production/stack.yml` from `project-name_stack/development/stack.yml` and [deploy.sh](https://gist.github.com/Dargmuesli/6f303f4550b8ff241897dbda30a49cb3) for automatic deployment.
