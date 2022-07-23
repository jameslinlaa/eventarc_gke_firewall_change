# Trigger cloud run app to handle firewall change event when changing service port on GKE 
This is an sample cloud run app with eventarc to handle GKE firewall change events. 
Ideally, the app can be implemented like notifing IT admin when someone apply new/modified service to GKE cluster and open/change some network ports (new/modified firewall rules)   \
In this sample code, I just print the content out in the log, but it can be integreated with other IM like slack or others in the real world case. \
The idea of this one is similar to the official sample 
[Receive a Cloud Audit Logs event](https://cloud.google.com/eventarc/docs/run/cal)
In this case, we focus on the audit log event when applying a yaml file in GKE and trigger firewall event changed (like change ingress port from 30000 to 30001)

# High-level Flow
1. Create a gke cluster, and deploy an ingress service 
2. Build the golang sample via cloud build
3. Deploy the event receiver service to Cloud Run 
4. Create an Eventarc trigger
5. Apply an updated ingress yaml file (fire a fw audit log)


# Build the golang sample

```
gcloud builds submit --tag gcr.io/<PROJECT_ID>/firewall-events
```

# Deploy the event receiver service to Cloud Run 

```
gcloud run deploy firewall-events \
   --image gcr.io/<PROJECT_ID>/firewall-events \
    --allow-unauthenticated
```


# Create an Eventarc trigger

Be careful of the "location" if your cloud run service location is not in us-central1, please set eventarc with "global". \
Please refer to [Known issue](https://cloud.google.com/eventarc/docs/creating-triggers#:~:text=Note%3A%20There,or%20global)

```
gcloud eventarc triggers create fw-events-trigger \
      --destination-run-service=firewall-events \
      --destination-run-region=asia-east1 \
      --event-filters="type=google.cloud.audit.log.v1.written" \
      --event-filters="serviceName=compute.googleapis.com" \
      --event-filters="methodName=v1.compute.firewalls.update" \
      --location=global \
      --service-account=<PROJECT_NUMBER>-compute@developer.gserviceaccount.com
```


# Update a ingress with a different port
Apply an updated ingress yaml file (to trigger a fw audit log)

