
# Purpose

# Scope

# Best Practices

---

## 1. Executive Summary

## 2. The Problem

## 3. The History

* How did we get here?
* What are the milestones and events that brought us to this point?

## 4. The Solution

* Build a process to..

## 5. The Benefits

* High confidence in confidentiality, integrity, and availability
* Trust is built on Integrity

## 6. The Call-To-Action

## 7. About "Company"


# Planning Outline: "what makes a hardened Linux system"

**Generic version** - not vendor specific, will be collaborating on this with SUSE and Canonical. This version will become the basis for a RHEL-specific version developed by Red Hatters. This will also be used as the basis for the RHEL Hardening Guide and the current version of the Guide can be used to help with this document.

What is ok to talk about in the generic version?

What is not ok to talk about in the generic version and should be reserved for the RHEL-specific version once we get there?

RHEL 8 Hardening Guide: [Red Hat Enterprise Linux 8 Security hardening](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/pdf/security_hardening/Red_Hat_Enterprise_Linux-8-Security_hardening-en-US.pdf)

Common Criteria: [Red Hat Adds Common Criteria Certification for Red Hat Enterprise Linux 8](https://www.redhat.com/en/about/press-releases/red-hat-adds-common-criteria-certification-red-hat-enterprise-linux-8)

1. Goal:
   1. The Compliance Collaborative for Enterprise Linux(s) wants to define a paradigm for “what constitutes a hardened Linux system.” In order to be able to define how to harden and secure these systems, consensus is needed to define deployment methods and security concerns pertaining to those environments. This outline will seek to define, with alignment to NIST SP 800-53 Control Families:
      1. Where Linux OS are deployed (on prem, disconnected, cloud, etc)
      1.  Specific challenges unique to those deployments (HSM, air-gap, virtualizations, etc)
   1. With a goal of determining how those challenges impact or determine how the OS will mitigate risk in those environments.
   1. From this outline, The Enterprise Linux providers will take the information gathered and use it to apply it to current in-house security technologies, features and products to achieve a hardened Linux system (OS or platform).
1. Deployment Types:
   1. Bare-Metal
   1. Virtualized
   1. Infrastructure Platform (Hypervisor)
1. Deployment Environments
   1. On-Prem
      1. Disconnected (air-gapped)
      1. DMZ
      1. Intranet
   1. Cloud
      1. Hyperscalers
      1. Workload
   1. Topology (past OS only concerns) - how they interact with Hybrid Solutions
1. What a hardened Linux System does/does not do (Next to define)
   1. What is a hardened Linux?
   1. Does:
      * Defaults to Secure Posture
      * Provide guidance on resiliency
      * Easily capable of managing certificates and implemented by default
      * Validate signatures of kernel modules prior to boot
      * Use strong, trusted cryptography starting at install time
      * Strong non-repudiation
      * Confirmed source of packages (like RH produced manifests)
      * Minimal application and package set for operation
      * Ensures trusted/secure boot
      * Allow list, applications, library, compilers, intrusion detection
   1. Does not do:
      * Unaltered universal image
      * Systems not reviewed or scanned for vulnerabilities
      * Unable to prove integrity
      * Out of date/unpatched system (unsupported; EOL)
      * No enterprise security integration
      * Logging disabled
      * No firewall enabled
      * Weak access controls
      * Do not uniquely identify users
      * Lack of good time source
      * Supported known compromised encryption protocols
      * Manual efforts; not automated
1. Continually Maintain

# Vendor Content

Describe process for vendors to create and sign their content (ex, packages, base images, etc) for Consumers to use to build their Organization base images

- Package manifests
- Checksums
- GPG signatures
- Vendor support contract to provide package updates and ongoing maintenance

# Consumer Organization Base Image(s)

This process should be automated and continually run on a recurring time basis as well as ad hoc as needed.

This process is the same whether building an organization's base image, ex just Linux with minimum packages, or a layered image such as a container, a specific piece of middleware, or a mission application layered on top of the base image or a layered image.

1. pull install content or pre-made image from Trusted source
1. pull additional content required to be installed on top of base install or base pre-made image to meet mission or hardening requirements
1. verify pulled content or pre-made image signatures are signed with trusted keys
1. if doing fresh install, install from content pulled from Trusted source signed with trusted keys
1. install any additional required content from Trusted source signed with Trusted keys
1. configure system has per hardening guidelines/requirements
1. scan system for vulnerabilities, EX CVEs
1. scan system for compliance checks
1. prepare installed system for templatization, ex
   1. wipe hostname, network configuration, any other machine identifying info
1. turn system into templatized image
1. create test system from new image
1. scan test system for vulnerabilities, EX CVEs
1. scan test system for compliance checks
1. run any mission specific functionality tests against test system
1. delete test system
1. publish template
1. sign template
1. notify dependent layered template jobs and runtime systems of new base image to kick off their release engineering process
1. Verify Logging in enabled and creating logs
1. Regularly confirmed backed up
1. Data at rest enabled
1. Data in motion - validated encryption
1. Data Loss prevention
1. Access control validation and maintenance (RBAC and ABAC)
1. Enterprise defined User accounts (Central and standalone)

# Consumer Organization Runtime System(s) Integration and Deployment

**DOING** - describe process for Consumer Organization to continually integrate runtime systems on top of base images and deploy them

- See To Do section above

### Summary of our Hardened Delivery model

#### Phase 1: (isolated provisioning zone)

* Kickstarted from immutable media (RHEL DVD downloaded, checksum validated, scanned)
*  SELinux enforcing
*  Firewall enforcing, add 22 (and 4750 for us)
*  FIPS=1
*  Partitions per stig and sanity (/tmp, /var, /opt, /var/log, /var/log/audit, /home ...)
*  Software added/removed for STIG from ideally minimal install.
*  Kdump killed
*  Postinstall script run for enterprise agent install (bladelogic)

At this point, as a vm template, the template would be cut here and phase 2 would start on vmware deploy triggers. For a physical blade, phase 2 just continues the process

#### Phase 2: (isolated provisioning zone)

* Automation properties populated and pushed for network zone, tenant/business system, etc including metadata on ownership, function, known accepted security exceptions
* Compliance as Code current “accepted” release (we made minor changes) pushed out to server (content, custom toolset) with other defined required files including enterprise-wide tailoring files.

Patch Server from current repo, reboot

Deployed by Template:

* Enterprise-mandated software suites deployed to server (McAfee HBSS, Netbackup)
  * Deployments tailored based on network zone properties
* Customized SELinux packages pushed for enterprise and/or business software
* Enterprise and business-system firewall rules
* Standard network customizations for DNS, Time, mail gateway, etc

Deployed by ansible:

* FAPolicyD enterprise base configuration
* AIDE entreprise base configuration
* A few other random bits

Hardening pass 1:

Execute oscap run against compliance as code content with os-specific tailoring file. Custom python integrated zone, system, etc exceptions into master tailoring file for server and validates during runtime. Run applies all STIG’s present in content_profile_stig with changes noted in tailoring.

Hardening pass 2:

Anything not found in the CAC content got written and dumped into a master ansible yaml and included in pass 2… last check, i think there 138. Ansible was configured to use exception logic fed into hardening pass 1 as well for a unified approach.

Patch server, reboot

Install phase automation steps complete. VM or server shut down, and moved to either destination service zone (non-dmz) or dmz install zone (dmz with explicit reachback to software repos)

Phase 3: “Destination Zone”

* More automation logic, then…
* Add centrally managed user accounts and ssh-rsa public keys
* Install and configure zone-dependent software (Solarwinds, others)
* Patch and reboot (in case physical delay in hardware moves)
* Kick off SCC, OSCAP, and ACAS scans.
* Publish scan results to file share

Build complete.

So going through this in my mind, i see some things the jump right out at me as adding value to “hardened”

* Validated immutable media at the source
* Cryptographically validated patch sources
* FIPS encryption validation and enforcement from the initial boot
* Automated scan and remediation content
* Openly published compliance available to system owners
* Deliberate, well conceived security plan
* Compliant by Delivery, non-compliant by explicit administrative action
* Full non-repudiation (All those lovely audit STIGs automatically applied...)

### Consumer Organization Runtime Systems Monitoring

**Organization** describes processes for:

* Consumer Organization to continually scan runtime systems
* for new static vulnerabilities (CVEs),
* static compliance drift (ex, comparisons against STIG)
* life intrusion events.
* Additionally scanning Consumer Organization Base Images at rest for static vulnerabilities and static compliance drift to mark as stale as new vulnerabilities and compliance criteria are released and old images become vulnerable.
* Continuous observability and monitoring

