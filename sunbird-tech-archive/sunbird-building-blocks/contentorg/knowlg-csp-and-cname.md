---
icon: elementor
---

# Knowlg - CSP & CNAME

**CNAME**: _CNAME_ records can be used to alias one name to another. _CNAME_ stands for Canonical Name.

All the Data in the DB & response should be CSP agnostic.

Presently the data stored in DB & API response is having the CSP provider details. Presently we are storing the absolute URL of the the assest in the DB & the same is sending in the response.

### Impact areas: <a href="#knowlg-csp-and-cname-impactareas" id="knowlg-csp-and-cname-impactareas"></a>

* Plugins loading in editors & packing as part of ECAR.
* ~~Generation of pre-signed URL used by Editors~~
* ECAR files will be having absolute paths (Need discussion on how to solve)
*   Data stored in the DB’s

    * downloadUrl
    * variants

    \----------------

    * previewUrl
    * appIcon
    * posterImage
    * artifactUrl
    * toc\_url
    * thumbnail
    * assetMap

#### Example: <a href="#knowlg-csp-and-cname-example" id="knowlg-csp-and-cname-example"></a>

[https://ntpproductionall.blob.core.windows.net/ntp-content-production/content/assets/do\_31329674183946240014126/a-boy-animation-1.mp4](https://ntpproductionall.blob.core.windows.net/ntp-content-production/content/assets/do\_31329674183946240014126/a-boy-animation-1.mp4)

### Data for CSP migration: <a href="#knowlg-csp-and-cname-dataforcspmigration" id="knowlg-csp-and-cname-dataforcspmigration"></a>

[https://docs.google.com/spreadsheets/d/13DaXCx8uToOwinlAPxvTat8NELxiPgG4KXATcKaJm\_c/edit?usp=sharing](https://docs.google.com/spreadsheets/d/13DaXCx8uToOwinlAPxvTat8NELxiPgG4KXATcKaJm\_c/edit?usp=sharing)

### CSP flow diagram to migrate the data: <a href="#knowlg-csp-and-cname-cspflowdiagramtomigratethedata" id="knowlg-csp-and-cname-cspflowdiagramtomigratethedata"></a>

[https://drive.google.com/file/d/1HzUob6a9\_TrVWhcoo1nWoKPVN0PKzpmQ/view?usp=share\_link](https://drive.google.com/file/d/1HzUob6a9\_TrVWhcoo1nWoKPVN0PKzpmQ/view?usp=share\_link)

### Solution: <a href="#knowlg-csp-and-cname-solution" id="knowlg-csp-and-cname-solution"></a>

Don’t store the absolute paths in the DB’s. Always store the path with CNAME as prefix.

example:

[https://store.diksha.gov.in/ntp-content-production/content/assets/do\_31329674183946240014126/a-boy-animation-1.mp4](https://store.diksha.gov.in/ntp-content-production/content/assets/do\_31329674183946240014126/a-boy-animation-1.mp4)

Configure the service with CNAME value as shown below

cname:\[\
{\
"type": "blob",\
"cname": "[https://store.diksha.gov.in",](https://store.diksha.gov.in",)\
"default": "[https://ntpproductionall.blob.core.windows.net/](https://ntpproductionall.blob.core.windows.net/) ",\
"properties": \[\
"downloadUrl",\
"artifactUrl",\
"streamingUrl",\
"posterImage",\
"previewUrl"\
]\
},\
{\
"type": "cdn",\
"cname": "[https://cdn.diksha.gov.in",](https://cdn.diksha.gov.in",)\
"default": "[https://cdn-ntpproduction.core.windows.net/](https://cdn-ntpproduction.core.windows.net/) "\
}\
]

cname: Value to be used to replace

default: Value that has to be replaced with CNAME

**properties**: Service will be looking for these properties while writing. Any aboslute path sending for these properties will get updaetd with CNAME value

Only while creation(POST API calls), we will be replacing the properties on the request body(CNAME\_properties) with CNAME & store in the DB.

The list of CNAME\_properties can be configured by any adopters as per their schema properties.

#### Challenges: <a href="#knowlg-csp-and-cname-challenges" id="knowlg-csp-and-cname-challenges"></a>

* We can handle only CNAME as part of this scope. Any domain name changes requires data migration as the DB stores the full path with CNAME+Domain.\
  Note: This can be generalised by handling the CNAME+Domain on service side while sending the responses by storing only relative paths in the DB.
* ECAR is packaged with CNAME values. So any domain change required data migration by republishing the ECAR’s(Content republish).
* StreamingUrl will required the data migration as it is specific to CSP providers. This may requires regeneration of all streaming URL’s with new CSP provider.
*

JOB Files to be addressed:\
asset-enrichment -> org.sunbird.job.assetenricment.helpers.ImageEnrichmentHelper -> enrichImage method -> optimizeImage method

asset-enrichment -> org.sunbird.job.assetenricment.helpers.OptimizerHelper -> replaceArtifactUrl method

asset-enrichment -> org.sunbird.job.assetenricment.helpers.VideoEnrichmentHelper ??

publish-pipeline -> content-publish -> org.sunbird.job.content.publish.helpers.ExtractableMimeTypeHelper -> getCloudStoreURL, updatePreviewUrl, getObjectWithEcar methods

publish-pipeline -> content-publish -> org.sunbird.job.content.publish.helpers.ContentPublisher -> updatePreviewUrl, getObjectWithEcar methods

publish-pipeline -> content-publish -> org.sunbird.job.content.publish.helpers.CollectionPublisher -> getObjectWithEcar method
