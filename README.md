# Ansible Role: NSS Shared DB

[![Build Status](https://travis-ci.org/pixelart/ansible-role-nssdb.svg?branch=master)](https://travis-ci.org/pixelart/ansible-role-nssdb)

Installs CA certificates into NSS Shared DB on Ubuntu or Debian, like Google Chrome uses it.

## Requirements

  - The CA certificate should be already installed on the target host. You can use `bdellegrazie.ca-certificates` for that as you still need that for curl and for PHP.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    certutils_package_state: installed

By default, this is set to 'installed'. You can override this variable to 'latest' if you want to upgrade or have switched to a different repo.

    nssdb_users: []
    
Add a list of user account names for which the certificates should be managed too, beside system-wide `/etc/pki/nssdb`. This is needed, e.g for Google Chrome which only opens the users nssdb :sob:

    nssdb_certificates: []

Add a list of certificates to install into `/etc/pki/nssdb` with a `name`, `path` and `trust` trust attributes (See `certutil -H -A` for help of the `-t trustargs` parameter), e.g. `CT,c,c` for trust on signing websites (like Chrome need it), or `CT,C,C` to trust on signing websites, S/MIME (mail) certificates and code signing.

Organize your cert name as `cert common name - cert organization` so Chrome can show it neatly

    nssdb_certificates:
      # Install CAcert CA and trust on websites, S/MIME and code signing.
      - name: CA Cert Signing Authority - Root CA
        path: /usr/local/share/ca-certificates/cacert.crt
        trust: CT,C,C
      # Install corporate CA and trust only on websites
      - name: Acme CA - Acme Corp
        path: /usr/local/share/ca-certificates/acme.crt
        trust: CT,c,c

## Dependencies

None, but you can use `bdellegrazie.ca-certificates` to transfer the certificates on the target host and install them for curl, php and so on too.

## Example Playbook

    - hosts: phpdevs
    
      pre_tasks:
        - name: Download CA Cert Signing Authority
          uri:
            url: 'http://www.cacert.org/certs/root.crt'
            return_content: true
          register: cacert_pem
    
      vars_files:
        - vars/main.yml
        
      roles:
        - bdellegrazie.ca-certificates
        - pixelart.nssdb
        
*Inside `vars/main.yml`*:

    ca_certificates_trusted:
      - { pem: "{{ cacert_pem.content }}", name: cacert }
      - { pem: "{{ lookup('file', 'files/ssl/acme-ca.pem') }}", name: acme }

    nssdb_users: ['username']
    nssdb_certificates:
      - name: CA Cert Signing Authority - Root CA
        path: '{{ ca_certificates_local_dir }}/cacert.crt'
        trust: CT,C,C
      - name: Acme CA - Acme Corp
        path: /usr/local/share/ca-certificates/acme.crt
        trust: CT,c,c

After the playbook runs the certificates are installed in the system-wide and users nssdb and also concatenated into the `ca-certficates.crt` for curl, php and so on.

## License

MIT, see the [LICENSE](LICENSE) file.

## Author Information

This role was created in 2017 by [pixelart GmbH](https://www.pixelart.at/).
