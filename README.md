# CREATE AAD GROUP USING AZ DEVOPS

Greetings to my fellow Technology Advocates and Specialists.

In this Session, I will demonstrate __How to Create Azure Active Directory (AAD) Group Using Azure DevOps.__

I had the Privilege to talk on this topic in __TWO__ Azure Communities:-

| __NAME OF THE AZURE COMMUNITY__ | __TYPE OF SPEAKER SESSION__ |
| --------- | --------- |
| __Journey to the Cloud 9.0__ | __Virtual__ |
| __Festive Tech Calendar 2022__ | __Virtual__ |


| __LIVE RECORDED SESSION:-__ |
| --------- |
| __LIVE DEMO__ was Recorded as part of my Presentation in __JOURNEY TO THE CLOUD 9.0__ Forum/Platform |
| Duration of My Demo = __55 Mins 42 Secs__ |
| [![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/EGIOzEpOxzE/0.jpg)](https://www.youtube.com/watch?v=EGIOzEpOxzE) |
| __LIVE DEMO__ was Recorded as part of my Presentation in __FESTIVE TECH CALENDAR 2022__ Forum/Platform |
| Duration of My Demo = __1 Hour 05 Mins 08 Secs__ |
| [![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/pcIVKO2dlEI/0.jpg)](https://www.youtube.com/watch?v=pcIVKO2dlEI&t=80s) |


| __IMPORTANT NOTE:-__ |
| --------- |
| __We can create one or more AAD Group with Same Name. The Unique Identifier for AAD Group is the Object ID.__|


| __USE CASE:-__ |
| --------- |
| Cloud Engineer __DOES NOT__ have access to __Azure Active Directory__ to Create Group(s). |
| Cloud Engineer __CANNOT ELEVATE__ rights using __PIM (Privileged Identity Management)__to Create AAD Group(s). |


| __AUTOMATION OBJECTIVE:-__ |
| --------- |
| Validate If the AAD Group Exists. If __Yes__, Pipeline will __FAIL__. |
| If the above validation is __SUCCESSFUL__, Pipeline will then Create Group in Azure Active Directory. |


| __IMPORTANT NOTE:-__ |
| --------- |
The YAML Pipeline is tested on __WINDOWS BUILD AGENT__ Only!!!


| __REQUIREMENTS:-__ |
| --------- |

1. Azure Subscription.
2. Azure DevOps Organisation and Project.
3. Service Principal either assigned Global Administrator, Privileged Identity Management (PIM) Azure AD Role or Required Microsoft Graph API Rights.(__Directory.ReadWrite.All__: Read and Write Directory Data). 
4. Azure Resource Manager Service Connection in Azure DevOps.


| __HOW DOES MY CODE PLACEHOLDER LOOKS LIKE:-__ |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/plx83e6s22oyl9811s14.png) |

| PIPELINE CODE SNIPPET:- | 
| --------- |

| AZURE DEVOPS YAML PIPELINE (azure-pipelines-add-single-aad-group-v1.0.yml):- | 
| --------- |

```
trigger:
  none

######################
#DECLARE PARAMETERS:-
######################
parameters:
- name: SubscriptionID
  displayName: Subscription ID Details Follow Below:-
  type: string
  default: 210e66cb-55cf-424e-8daa-6cad804ab604
  values:
  - 210e66cb-55cf-424e-8daa-6cad804ab604

- name: AADGRPNAME
  displayName: Please Provide the AAD Group Name:-
  type: object
  default: 

######################
#DECLARE VARIABLES:-
######################
variables:
  ServiceConnection: amcloud-cicd-service-connection
  BuildAgent: windows-latest

#########################
# Declare Build Agents:-
#########################
pool:
  vmImage: $(BuildAgent)

###################
# Declare Stages:-
###################

stages:

- stage: CREATE_SINGLE_AAD_GROUP 
  jobs:
  - job: CREATE_SINGLE_AAD_GROUP 
    displayName: CREATE SINGLE AAD GROUP
    steps:
    - task: AzureCLI@2
      displayName: VALIDATE AND CREATE AAD GROUP
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show
          $name = az ad group show --group ${{ parameters.AADGRPNAME }} --query "displayName" -o tsv
          if ($name -eq "${{ parameters.AADGRPNAME }}") {
          echo "################################################################################################"
          echo "Azure AD Group ${{ parameters.AADGRPNAME }} EXISTS and hence Cannot Proceed with Creation!!!"
          echo "################################################################################################"
          exit 1
          }
          else {
          echo "############################################################################"
          echo "THE ABOVE WARNING IS A STANDARD MESSAGE WHEN AAD GROUP DOES NOT EXISTS!!!"
          echo "AAD GROUP BY THE NAME ${{ parameters.AADGRPNAME }} WILL BE CREATED"
          echo "############################################################################"
          az ad group create --display-name ${{ parameters.AADGRPNAME }} --mail-nickname ${{ parameters.AADGRPNAME }}	
          echo "##################################################################"
          echo "Azure AD Group ${{ parameters.AADGRPNAME }} created successfully!!!"
          echo "##################################################################"
          }

```

__Now, let me explain each part of YAML Pipeline for better understanding.__

| __PART #1:-__ | 
| --------- |

| __BELOW FOLLOWS PIPELINE RUNTIME VARIABLES CODE SNIPPET:-__ | 
| --------- |

```
######################
#DECLARE PARAMETERS:-
######################
parameters:
- name: SubscriptionID
  displayName: Subscription ID Details Follow Below:-
  type: string
  default: 210e66cb-55cf-424e-8daa-6cad804ab604
  values:
  - 210e66cb-55cf-424e-8daa-6cad804ab604

- name: AADGRPNAME
  displayName: Please Provide the AAD Group Name:-
  type: object
  default: 

```

| __PART #2:-__ | 
| --------- |

| __BELOW FOLLOWS PIPELINE VARIABLES CODE SNIPPET:-__ | 
| --------- |

```
######################
#DECLARE VARIABLES:-
######################
variables:
  ServiceConnection: amcloud-cicd-service-connection
  BuildAgent: windows-latest

```

| __NOTE:-__ | 
| --------- |
| Please change the values of the variables accordingly. |
| The entire YAML pipeline is build using __Runtime Parameters and Variables__. No Values are Hardcoded. |


| __PART #3:-__ | 
| --------- |

| __BELOW FOLLOWS THE CONDITIONS AND LOGIC DEFINED IN THE PIPELINE (AS MENTIONED ABOVE IN THE "AUTOMATION OBJECTIVE"):-__ | 
| --------- |

```
inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show
          $name = az ad group show --group ${{ parameters.AADGRPNAME }} --query "displayName" -o tsv
          if ($name -eq "${{ parameters.AADGRPNAME }}") {
          echo "################################################################################################"
          echo "Azure AD Group ${{ parameters.AADGRPNAME }} EXISTS and hence Cannot Proceed with Creation!!!"
          echo "################################################################################################"
          exit 1
          }
          else {
          echo "############################################################################"
          echo "THE ABOVE WARNING IS A STANDARD MESSAGE WHEN AAD GROUP DOES NOT EXISTS!!!"
          echo "AAD GROUP BY THE NAME ${{ parameters.AADGRPNAME }} WILL BE CREATED"
          echo "############################################################################"
          az ad group create --display-name ${{ parameters.AADGRPNAME }} --mail-nickname ${{ parameters.AADGRPNAME }}	
          echo "##################################################################"
          echo "Azure AD Group ${{ parameters.AADGRPNAME }} created successfully!!!"
          echo "##################################################################"
          }

```

__NOW ITS TIME TO TEST !!!...__

| __TEST CASES:-__ | 
| --------- |

| __TEST CASE #1: AAD GROUP EXISTS:-__ | 
| --------- |
| __DESIRED OUTPUT: PIPELINE FAILS STATING THAT THE MENTIONED AAD GROUP EXISTS.__ |
| __AAD GROUP IN PLACE:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gr553uovtet2mdunwl9w.jpg) |
| __PIPELINE RUNTIME VARIABLES VALUE:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/29whhujlb5ckx0z0gayu.jpg) |
| __PIPELINE FAILED:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zfvq9nk4g3xdysm3zutm.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vhxqme5oq86wnw0bhe87.png) |
| __TEST CASE #2: AAD GROUP DID NOT EXISTS:-__ |
| __DESIRED OUTPUT: PIPELINE EXECUTED SUCCESSFULLY CREATING THE AAD GROUP.__ |
| __PIPELINE EXECUTED SUCCESSFULLY:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2zkl18cu50qksq6f5cxj.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i5uwmk6ng7fioijphngc.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hmj21u3t6nm6turcb3oc.jpg) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hts18zjj45908xlycc1h.jpg) |


__Hope You Enjoyed the Session!!!__

__Stay Safe | Keep Learning | Spread Knowledge__
