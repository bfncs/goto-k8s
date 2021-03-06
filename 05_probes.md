## Liveness & Readiness

---

### What is Liveness and Readiness?

Kubernetes health checks are divided into liveness and readiness probes. 

Kubernetes is focusing on running containers in production. Production means that we need a way to ensure pods are actually running and healthy.

---

### Readiness Probe

Readiness probes allow you to specify checks to verify if a Pod is ready for use. There are three methods that can be used to determine readiness. HTTP, Exec or TCPSocket.

```
readinessProbe:
  httpGet:
    path: /readiness
    port: 8080
  initialDelaySeconds: 5  # Delay before probe is called
  timeoutSeconds: 1       # Probe must respond within this timeout
```
Response must be HTTP Status Code between 200 and 399

---

### Liveness probe

Once the application pod is up we need to confirm that it’s healthy and ready for serving traffic.

Liveness probes are for situations when an app has crashed or isn't responding anymore.

```
livenessProbe:
  httpGet:
    path: /healthcheck
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10        # Will be called every 10 seconds
  timeoutSeconds: 1        # Probe must respond within 1 second 
```

---

### Liveness Probes

As a simple example here is a health for a Go applications.

```
http.HandleFunc("/healthcheck", func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("ok"))
}
http.ListenAndServe(":8080", nil)
```

---

And this needs to be added into the Pod manifest
```
livenessProbe:
  httpGet:
    path: /healthcheck
    port: 8080
  initialDelaySeconds: 15
  timeoutSeconds: 1
```

---

### Readiness Probes

Can check other pieces, such as a database connection

```
http.HandleFunc("/readiness", func(w http.ResponseWriter, r *http.Request) {
  ok := true
  errMsg = ""

  // Check database
  if db != nil {
    _, err := db.Query("SELECT 1;")
  }
  if err != nil {
    ok = false
    errMsg += "Database not ok.\n"
  } 

  if ok {
    w.Write([]byte("OK"))
  } else {
    // Send 503
    http.Error(w, errMsg, http.StatusServiceUnavailable)
  }
})
http.ListenAndServe(":8080", nil)
```

---

And this needs to be added into the Pod manifest

```
readinessProbe:
  httpGet:
    path: /readiness
    port: 8080
  initialDelaySeconds: 20
  timeoutSeconds: 5
```

Combining the readiness and liveness probes help ensure only healthy containers are running within the cluster. With the liveness probe you can monitor also downstream dependencies.

---

### Advanced liveness probe example

```
livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
          - name: X-Custom-Header
            value: Awesome
      initialDelaySeconds: 15
      timeoutSeconds: 1
```

`httpHeaders` describes a custom header to be used in HTTP probes

---

### Creating Pods with Liveness and Readiness Probes

Explore the healthy-monolith pod configuration:

```
...
      livenessProbe:
        httpGet:
          path: /healthz
          port: 81
          scheme: HTTP
        initialDelaySeconds: 5
        periodSeconds: 15
        timeoutSeconds: 5
      readinessProbe:
        httpGet:
          path: /readiness
          port: 81
          scheme: HTTP
        initialDelaySeconds: 5
        timeoutSeconds: 1
```

---

Create the healthy-monolith pod using 
```
kubectl create -f ./resources/healthy-monolith.yaml
```

Thanks to Kelsey Hightower for this application

---

### View Pod details

Pods will not be marked ready until the readiness probe returns an HTTP 200 response. Use the `kubectl describe` to view details for the healthy-monolith Pod.

The healthy-monolith Pod logs each health check. Use the `kubectl logs` command to view them.

---

### Experiment with Readiness Probes

Let's see how Kubernetes handels failed readiness probes. 

The monolith container supports the ability to force failures of it's readiness and liveness probes (again thanks to Kelsey)

---

Use the `kubectl port-forward` command to forward a local port to the health port of the healthy-monolith Pod.

```
kubectl port-forward healthy-monolith 10081:81
```
You now have access to the /healthz and /readiness HTTP endpoints

---

Force the monolith container readiness probe to fail. Use the curl command to toggle the readiness probe status:

```
curl http://127.0.0.1:10081/readiness/status
```
Wait about 45 seconds and get the status of the healthy-monolith Pod using the kubectl get pods command:

```
kubectl get pods healthy-monolith
```

---

Use the kubectl describe command to get more details about the failing readiness probe:

```
kubectl describe pods healthy-monolith
```

Notice the details about failing probes.

```
Liveness:     http-get http://:81/healthz delay=5s timeout=5s period=15s #success=1 #failure=3
Readiness:    http-get http://:81/readiness delay=5s timeout=1s period=10s #success=1 #failure=3
```

---

Force the monolith container readiness probe to pass. Use the curl command to toggle the readiness probe status:

```
curl http://127.0.0.1:10081/readiness/status
```
If we use the `--watch` (`-w`) flag we can see when the status of the pod changes:

```
kubectl get pods healthy-monolith -w
```

---

### Experiment with Liveness Probes

Building on what you learned in the previous tutorial use the kubectl port-forward and curl commands to force the monolith container liveness probe to fail. Observe how Kubernetes responds to failing liveness probes.

```
curl http://127.0.0.1:10081/healthz/status
```

---

### Quiz

* What happened when the liveness probe failed?
* What events where created when the liveness probe failed?

---


### Cleanup

```
kubectl delete -f ./resources/healthy-monolith.yaml
```

---

### Exercise - Add Probes for Deals microservice

* Add Readiness probe to the Deals deployment
 (hint: endpoint already exists)
* Deploy the updated config
* Add Liveness probe to the Deals deployment
* Deploy the updated config

---

[Next up Resources](../06_resources.md)

