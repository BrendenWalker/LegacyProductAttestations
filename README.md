# About the author - Brenden Walker

I have been professionaly employed in technology for about 40 years. Extensive experience with network engineering, cybersecurity and software engineering allows me to approach this type of work in a way that balances the needs of IT, Management and Engineering. 

Education includes a BS in Cybersecurity and multiple ISC2 certifications, notably [ISC2 CSSLP](https://www.credly.com/badges/5a1765cd-b937-478c-95e3-fe4a56620031/public_url).

I have used the following techniques to successfully pass PA-DSS audits and as far as I know this is still in play at a former employer. Hopefully this helps others.

# Legacy Product Attestations
So you have a product written in a language without any commercial SAST support and customers are asking for SBOM's and/or Attestations.

This repo and these instructions will hopefully help.

## CISA Attestation Form
The U.S. Cybersecurity and Infrastructure Securty Agency has a well thought out [Attestation Form](https://www.cisa.gov/resources-tools/resources/secure-software-development-attestation-form)

Completion of this self-attestation form is the bare minimum required for U.S. Government agencies software procurement. I believe this would satisfy most commercial entities as well.

Sections I and II could be automatically generated by including the information in a file in the repository and some 'gentle massaging'.  Or just entered manually, it't not that much work.

Section III is where things get more interesting.  I'll go over the major points briefly:

> 1. The software is developed and built in secure environments. Those environments are secured by the following actions, at a minimum:

#1 is a fairly broad domain involving IT/Cybersec/Policy.  For now I'm leaving this out of scope as I'll be focusing on the challenges presented by legacy software.

> 2. [The software producer makes a good-faith effort to maintain trusted source code supply chains by employing automated tools or comparable processes to address the security of internal code and third-party components and manage related vulnerabilities;](#cisa-attestation-form-section-iii-2---supply-chain-security)

#2 is your your supply chain security.  

> 3. [The software producer maintains provenance for internal code and third-party components incorporated into the software to the greatest extent feasible;](#cisa-attestation-form-section-iii-3---code-provenance)

#3 is likely best served by producing and maintaining an SBOM (Software Bill Of Materials).

> 4. [The software producer employs automated tools or comparable processes that check for security vulnerabilities. In addition:](#cisa-attestation-form-section-iii-4---toolsprocesses-to-check-code-for-security-vulnerabilities)

#4 This is an area that 'legacy' code bases will struggle with. 

> 5. The software producer operates a vulnerability disclosure program and accepts, reviews, and addresses disclosed software vulnerabilities in a timely fashion and according to any timelines specified in the vulnerability

#5 is outside the scope of this project, for now.

The following sections will dig in deeper.

### CISA Attestation Form Section III (2) - Supply Chain Security

A good-faith effort employing automated tools OR comparable processes is possible.  Note that this covers both internal and third party components. 

Recommendations:
- Enforce required code reviews.
- SAST tooling, even if it doesn't directly support your language many will catch credentials and other potential concerns.
- Document all third party code. I simple method I have used with good results is:
    - All third party code is added to the product repo in product/component specific folder under a folder dedicated to third party code (lib, thirdparty, 3rdparty, etc).
    - Each third party code folder contains a file describing provonance. INI file, JSON, XML.. whatever you prefer.
    - Build a script that does the following:
        - Iterates all third party folders validating that all top level folders have the appropriate information file:
          - for each file found, load product info file.
          - Any folders that are missing product info are noted
          - Information is collected into a single file, customer format.. [cyclondx](https://cyclonedx.org/) whatever works for you.
              - NOTE: consider including a hash of the folder and all contents, this provides a high level of assurance that nothing has changed.
    - Check in an initial run of the above script.  This becomes your baseline.
    - Add the script to the CICD process, ** as early as possible **
      -  Any missing product info files need to break CICD.
      -  The generated file is compared to the checked in baseline, any differences are noted and the CICD failed.
      
**IF** your code base relies on third party components stored externally (npm, nuget or some other package repository) there will be additional work.  My recomendation in this area is to host your own repository and block access to public repos from CICD tooling. 

You can see the above has created more work when updating third party tools. Some things to consider:
- Subcribe to CVE alerts for each component.  Having a 'developers@company.com' or similar shared email can be helpful here.
- Assigning CODEOWNERS (or similar) to folders containing third party code can alert a responsible party of changes.
- Document the correct process of updating third party code. Try to make the process easy to follow.

### CISA Attestation Form Section III (3) - Code Provenance

Adding SBOM output to ad-hoc automated tooling that supports controlling supply chain security should be fairly trivial.  The standard [cyclonedx](https://cyclonedx.org/) SBOM file can be injested by many code hosting solutions, recommend implementing and integrating into CICD if possible.

Alternately, a CSV file or spreadsheet can be generated which may be good enough for attestation purposes. You need to show that you are controlling the supply chain, how specifically is up to you.

### CISA Attestation Form Section III (4) - Tools/Processes to check code for security vulnerabilities

Work for the prior sections will feed into this, and likely feed into new policy which supports your answers to this section. 

Implementing a 'vulnerability disclosure program' can be as simple as a published email address and policy/procedures to handle any disclosures.

# Managing Licensing Information

Including license information along with provenance is worth considering.  If you have ever been involved in due diligence for a acquisition you have likely met with a black duck scan. Having licensing information documented accuratly can go a long way toward showing good development practices.

# Appendix - Links that may be handy.

https://www.cisa.gov/resources-tools/resources/secure-software-development-attestation-form
https://slsa.dev/attestation-model
https://www.cisa.gov/news-events/news/cisa-publishes-repository-software-attestation-and-artifacts
https://www.lawfaremedia.org/article/making-attestation-work-for-software-security
https://www.cisa.gov/news-events/news/cisa-publishes-repository-software-attestation-and-artifacts
https://cyclonedx.org/
