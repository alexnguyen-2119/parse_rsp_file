# parse_rsp_file

Ansible role that parses a certificate response file and classifies exactly three certificates regardless of order:

- Issued certificate
- Signing CA certificate
- Root CA certificate

The role stores each classified certificate as PEM and writes a YAML summary file.

## Requirements

- `openssl` must be installed on the target host.
- Input file must exist and be non-empty.
- Input must contain (or resolve to) exactly 3 certificates.

## Role Variables

Defaults are defined in `roles/parse_rsp_file/defaults/main.yml`.

- `cert_bundle_rsp` (default: `all-certs.rsp`)
  - Path to the input response file.
- `output_file` (default: `parsed-certs.yml`)
  - Path to the generated YAML summary output.
- `local_work_dir` (default: `{{ playbook_dir }}/cert-work`)
  - Local working directory for intermediate files.
- `pem_output_dir` (default: `{{ playbook_dir }}/cert-work/pem`)
  - Output directory for classified PEM files.

## What the role does

1. Validates that `cert_bundle_rsp` exists and is non-empty.
2. Tries PKCS#7 extraction using `openssl pkcs7 -print_certs`.
3. Falls back to original source if extraction returns zero cert blocks.
4. Splits PEM certs into local files.
5. Reads subject/issuer and classifies certificates by chain relationship.
6. Stores:
   - `issued-certificate.pem`
   - `signing-ca-certificate.pem`
   - `root-ca-certificate.pem`
7. Writes summary YAML file (`output_file`) containing file paths and subjects.

## Outputs

Generated files in `pem_output_dir`:

- `issued-certificate.pem`
- `signing-ca-certificate.pem`
- `root-ca-certificate.pem`

Generated summary file (`output_file`) includes:

- `issued_certificate_file`
- `signing_ca_certificate_file`
- `root_ca_certificate_file`
- `issued_subject`
- `signing_ca_subject`
- `root_ca_subject`

## Example Playbook

```yaml
---
- name: Parse certificate response file
  hosts: localhost
  gather_facts: false

  roles:
    - role: parse_rsp_file
```

## Example with variable override

```yaml
---
- name: Parse certificate response file
  hosts: localhost
  gather_facts: false

  roles:
    - role: parse_rsp_file
      vars:
        cert_bundle_rsp: all-certs.pem
        output_file: parsed-certs.yml
```
