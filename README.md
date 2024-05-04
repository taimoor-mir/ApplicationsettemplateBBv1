# kubernetes-service-catalog
Agnostic kubernetes service catalog for all kubernetes clusters

#USE CASE - USE FOR OSS IMAGES TO KEEP THEM UP TO DATE THROUGH GH ACTION WHICH WILL
1.  CHECK FOR NEW VERSIONS OF CHARTS
2. CHANGE THE VERSION AND CREATE A PR 
3. WE'LL AUTO APPROVE, FOR NOW, THOSE PR WHICH WILL UPDATE THE VERSION NUMBER OF THE CHART
4. APPLICATIONSET WILL PICKUP CHANGE AND DEPLOY VIA ARGOCD 

#create applicationset(s) for use with
1. deploying as argocd cluster
2. appsec light stack
3. appsec dark stack

note: will use applicationset generator: list and cluster AND at least version 2.8 of Argo to deploy to
 SPECIFIC NAMESPACES ON A CLUSTER VERSUS THE DEFAULT DEPLOYMENT OF APPSETS WHICH ARE TYPICALLY DEPLOYED TO ARGOCD NAMESPACE


