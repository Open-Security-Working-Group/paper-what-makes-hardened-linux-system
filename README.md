# paper-what-makes-hardened-linux-system (Latest v2 Draft)
## Planning Outline: “what makes a hardened Linux system”

Generic version - not vendor specific, will be collaborating on this with SUSE and Canonical. This version will become the basis for a RHEL-specific version developed by Red Hatters. This will also be used as the basis for the RHEL Hardening Guide and the current version of the Guide can be used to help with this document.

What is ok to talk about in the generic version?

What is not ok to talk about in the generic version and should be reserved for the RHEL-specific version once we get there?

RHEL 8 Hardening Guide: Red Hat Enterprise Linux 8 Security hardening

Common Criteria: Red Hat Adds Common Criteria Certification for Red Hat Enterprise Linux 8

## Goal:

The Compliance Collaborative for Enterprise Linux(s) wants to define a paradigm for “what constitutes a hardened Linux system.” In order to be able to define how to harden and secure these systems, consensus is needed to define deployment methods and security concerns pertaining to those environments. This outline will seek to define, with alignment to NIST SP 800-53 Control Families:

Where Linux OS are deployed (on prem, disconnected, cloud, etc)

Specific challenges unique to those deployments (HSM, air-gap, virtualizations, etc)

With a goal of determining how those challenges impact or determine how the OS will mitigate risk in those environments.

From this outline, The Enterprise Linux providers will take the information gathered and use it to apply it to current in-house security technologies, features and products to achieve a hardened Linux system (OS or platform)

## Deployment Types:

Bare-Metal

Virtualized

Infrastructure Platform (Hypervisor)

## Deployment Environments

On-Prem

Disconnected (air-gapped)

DMZ

Intranet

Cloud

Hyperscalers

Workload

Topology (past OS only concerns) - how they interact with Hybrid Solutions

## What a hardened Linux System does/does not do (Next to define)

What is a hardened linux?

## Does:

Defaults to Secure Posture

Provide guidance on resiliency

Easily capable of managing certificates and implemented by default

Validate signatures of kernel modules prior to boot

Use strong, trusted cryptography starting at install time

Strong non-repudiation

Confirmed source of packages (like RH produced manifests)

Minimal application and package set for operation

Ensures trusted/secure boot

Allow list, applications, library, compilers, intrusion detection

## Does not do:

Unaltered universal image

Systems not reviewed or scanned for vulnerabilities

Unable to prove integrity

Out of date/unpatched system (unsupported; EOL)

No enterprise security integration

Logging disabled

No firewall enabled

Weak access controls

Do not uniquely identify users

Lack of good time source

Supported known compromised encryption protocols

Manual efforts; not automated

## Continually Maintain

## Vender Content

Describe process for vendors to create and sign their content (ex, packages, base images, etc) for Consumers to use to build their Organization base images

Package manifests

-Checksums

-GPG signatures

-Vendor support contract to provide package updates and ongoing maintenance

## Consumer Organization Base Image(s)

This process should be automated and continually run on a recurring time basis as well as ad hoc as needed.

This process is the same whether building an organization’s base image, ex just Linux with minimum packages, or a layered image such as a container, a specific piece of middleware, or a mission application layered on top of the base image or a layered image.

pull install content or pre-made image from Trusted source

pull additional content required to be installed on top of base install or base pre-made image to meet mission or hardening requirements

verify pulled content or pre-made image signatures are signed with trusted keys

if doing fresh install, install from content pulled from Trusted source signed with trusted keys

install any additional required content from Trusted source signed with Trusted keys

configure system has per hardening guidelines/requirements

scan system for vulnerabilities, EX CVEs

scan system for compliance checks

prepare installed system for templatization, ex

wipe hostname, network configuration, any other machine identifying info

turn system into templatized image

create test system from new image

scan test system for vulnerabilities, EX CVEs

scan test system for compliance checks

run any mission specific functionality tests against test system

delete test system

publish template

sign template

notify dependent layered template jobs and runtime systems of new base image to kick off their release engineering process

Verify Logging in enabled and creating logs

Regularly confirmed backed up

Data at rest enabled

Data in motion - validated encryption

Data Loss prevention

Access control validation and maintenance (RBAC and ABAC)

Enterprise defined User accounts (Central and standalone)

##Consumer Organization Runtime System(s) Integration and Deployment

DOING - describe process for Consumer Organization to continually integrate runtime systems on top of base images and deploy them

-See To Do section above

## Summary of our Hardened Delivery model:

# Phase 1: (isolated provisioning zone)

Kickstarted from immutable media (RHEL DVD downloaded, checksum validated, scanned)

SELinux enforcing

Firewall enforcing, add 22 (and 4750 for us)

FIPS=1

Partitions per stig and sanity (/tmp, /var, /opt, /var/log , /var/log/audit, /home…)

Software added/removed for STIG from ideally minimal install.

Kdump killed

Postinstall script run for enterprise agent install (bladelogic)

At this point, as a vm template, the template would be cut here and phase 2 would start on vmware deploy triggers. For a physical blade, phase 2 just continues the process

# Phase 2: (isolated provisioning zone)

Automation properties populated and pushed for network zone, tenant/business system, etc including metadata on ownership, function, known accepted security exceptions

Compliance as Code current “accepted” release (we made minor changes) pushed out to server (content, custom toolset) with other defined required files including enterprise-wide tailoring files.

Patch Server from current repo, reboot

# Deployed by Template:

Enterprise-mandated software suites deployed to server (McAfee HBSS, Netbackup)

Deployments tailored based on network zone properties

Customized SELinux packages pushed for enterprise and/or business software

Enterprise and business-system firewall rules

Standard network customizations for DNS, Time, mail gateway, etc

# Deployed by ansible:

FAPolicyD enterprise base configuration

AIDE entreprise base configuration

A few other random bits

Hardening pass 1:

Execute oscap run against compliance as code content with os-specific tailoring file. Custom python integrated zone, system, etc exceptions into master tailoring file for server and validates during runtime. Run applies all STIG’s present in content_profile_stig with changes noted in tailoring.

Hardening pass 2:

Anything not found in the CAC content got written and dumped into a master ansible yaml and included in pass 2… last check, i think there 138. Ansible was configured to use exception logic fed into hardening pass 1 as well for a unified approach.

Patch server, reboot

Install phase automation steps complete. VM or server shut down, and moved to either destination service zone (non-dmz) or dmz install zone (dmz with explicit reachback to software repos)

# Phase 3: “Destination Zone”

More automation logic, then…

Add centrally managed user accounts and ssh-rsa public keys

Install and configure zone-dependent software (Solarwinds, others)

Patch and reboot (in case physical delay in hardware moves)

Kick off SCC, OSCAP, and ACAS scans.

Publish scan results to file share

Build complete.

So going through this in my mind, i see some things the jump right out at me as adding value to “hardenned”

Validated immutable media at the source

Cryptographically validated patch sources

FIPS encryption validation and enforcement from the initial boot

Automated scan and remediation content

Openly published compliance available to system owners

Deliberate, well conceived security plan

Compliant by Delivery, noncompliant by explicit administrative action

Full nonrepudiation (All those lovely audit STIGs automatically applied…)

## Consumer Organization Runtime Systems Monitoring

Organization describes processes for:

Consumer Organization to continually scan runtime systems

for new static vulnerabilities (CVEs),

static compliance drift (ex, comparisons against STIG)

life intrusion events.

Additionally scanning Consumer Organization Base IMages at rest for static vulnerabilities and static compliance drift to mark as stale as new vulnerabilities and compliance criteria are released and old images become vulnerabile.

Continuous observability and monitoring *
