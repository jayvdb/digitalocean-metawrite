# DigitalOcean Meta-Write

dometawrite is a tool that helps system administrators take information from the DigitalOcean API and/or Dropleta Metadata API
to create text files such as configuration files, status pages or anything else that can be coded in a template.

Templates are written using the most-excellent Jinja2 library for Python.

# Example: Create SSH client configuration

dometawrite can be used to create an OpenSSH SSH Client configuration file that can be included from ~/.ssh/config for 
easy access to your droplets. Picture this:

    ~ $ dometawrite --template ssh-config  \
                    --api-key $DO_API_KEY  \
                    --output /home/example/.ssh/digitalocean_droplets \
                    -u user:remoteUser -u keyfile:/home/example/id_rsa_digitalocean

The above command will render the ssh-config dometawrite template, and write the output to the /home/example/.ssh/digitalocean_droplets file.

It will use the indicated DigitalOcean API KEY, and will also provide some additional variables to the template. If any variables
are missing from the command line, dometawrite will let you know.

The resulting output may look like this:

    Host droplet-name
    HostName droplet_ip_address
    User root
    IdentityFile /home/example/id_rsa_digitalocean
    
    Host another_droplet
    HostName yet_another_ip
    User root
    IdentityFile /home/example/id_rsa_digitalocean

As long as the id_rsa_digitalocean.pub file has been added to the droplet, either during creation or afterwards, then you will be able to
simply:

    ssh root@another_droplet

# Example: Create OpenSSH AuthorizedKeysFile-compatible output using from your DigitalOcean account.

This is a useful feature, but please do NOT --output directly to your authorized_keys file!

    dometawrite --template authorizedkeys \
                --api-key $DO_API_KEY

# Example: Create ansible inventory (This template is not yet available)

Maybe you use ansible, and you want to update your hosts inventory dynamically:

    dometawrite --template ansible-inventory \
                --api-key $DO_API_KEY        \
                --output /etc/ansible/hosts/digitaloceaninventory

The template will receive a python dictionary with all necessary information. Jinja2 supports advanced logic, so it can easily contain
all the required code to output a valid Ansible Inventory file.

# Templating Ideas

Templates will have to indicate endpoint requirements, and specific user variables.
For example, for the ssh-config template:

    {# Your first TWO(2) set commands MUST indicate endpoint_requirements and
    userdata_requirements #}
    {% set endpoint_requirements = ['droplets'] %}
    {% set userdata_requirements = ['user','keyfile'] %}

    {% for droplet in droplets.droplets %}
    Host {{ droplet.name }}
    HostName {{ droplet.networks.v4[0].ip_address }}
    User {{ userdata.user }}
    IdentityFile {{ userdata.keyfile }}
    
    {% endfor %}


Some endpoints include '/' in their name. The template-context variable name
is automatically named replacing all instances of '/' with '_'. For example,
take the authorizedkeys template:

    {% set endpoint_requirements = ['account/keys'] %}
    {% for key in account_keys.ssh_keys %}
    # Key "{{ key.name }}" ID={{ key.id }} Fingerprint={{key.fingerprint}}
    {{ key.public_key }}
    {% endfor %}

You can see that we iterate over each key using account_keys.ssh_keys as source,
but the endpoint_requirements is set to call 'accounts/keys' DO APIv2 endpoint.

# TODO

* add testing https://python-packaging.readthedocs.io/en/latest/testing.html
* add pagination support
