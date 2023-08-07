
###### tags: OLM v1 Migration

# OLM v1 Migration
### Descoped, cluster-wide singleton model for cluster extensions
- Only operator bundles with “AllNamespace” mode installation support can be lifecycled with the new APIs / flows in OLM 1.0
- Not going to remove OLM v0.x for as long as Openshift 4 exist
- When OLM v1.0 is ready, OLM v0.x will go into maintainace mode


--------

### OLM v0.x vs v1.x

![](https://hackmd.io/_uploads/SJArQ7_i3.png)

<div style="text-align: center;">
    <img src="images/olmv0-v1.png" height="200">
</div>



----
### Example of Namespaced-scoped vs Cluster-scoped operator

![](https://hackmd.io/_uploads/HJk2rX_oh.png)

<div style="text-align: center;">
    <img src="images/namespace-cluster.png" height="200">
</div>

- OLM v1, the Admin makes the operator accessible to tenants.

----

### Cluster-scoped operators Best-Practices

![](https://hackmd.io/_uploads/SJ-yvQdsn.png)

<div style="text-align: center;">
    <img src="images/bestpractice.png" height="200">
</div>


### - **Drop Assumptions of Multiple Operator Instances:**
    
   - Let's say you have an operator that deploys an application, AppX, and you have instances of this operator in multiple namespaces each deploying its own instance of AppX. This approach needs to be altered. In the cluster-scoped model, you'll have a single instance of your operator that manages all instances of AppX across all namespaces.

   - When moving from namespace-scoped to cluster-scoped, you must modify the manager setup in `main.go`. By default, it will manage resources in its own namespace:

   ```go
mgr, err := ctrl.NewManager(ctrl.GetConfigOrDie(), ctrl.Options{
    Scheme:             scheme,
    Namespace:          namespace,  // set to the operator's namespace
    MetricsBindAddress: metricsAddr,
    ...
})
   ```

   To make it cluster-scoped, just remove the `Namespace` option:

   ```go
mgr, err := ctrl.NewManager(ctrl.GetConfigOrDie(), ctrl.Options{
    Scheme:             scheme,
    MetricsBindAddress: metricsAddr,
    ...
})
   ```

### - **Reconcile every object within its namespace:**
   - In the namespace-scoped model, an operator watching a CustomResource (CR) would only react to changes in its own namespace. In a cluster-scoped operator, you'll change your watch function to observe changes in all namespaces.

   - For namespace-scoped operators, the reconciler would get the list of all instances of the custom resource within the operator's namespace. In `controllers/database_controller.go`, you might have something like:

   ```go
// Reconcile loop for Database
func (r *DatabaseReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    var databaseList v1alpha1.DatabaseList
    if err := r.List(ctx, &databaseList, client.InNamespace(req.Namespace)); err != nil {
        return ctrl.Result{}, err
    }
    // Handle each database instance in databaseList...
}
   ```
   
   For cluster-scoped, change to list with no namespace filter:

   ```go
func (r *DatabaseReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    var databaseList v1alpha1.DatabaseList
    if err := r.List(ctx, &databaseList); err != nil {
        return ctrl.Result{}, err
    }
    // Handle each database instance in databaseList...
}
   ```


### - **Read/store configuration or credentials only in the namespace in which the request came from:**
   - If your operator uses a `Secret` to store credentials, ensure that it reads the `Secret` from the namespace of the `Database` custom resource rather than the operator's namespace. Change:

```go
func (r *DatabaseReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // Assume secret is in operator's namespace
    secret := &corev1.Secret{}
    if err := r.Get(ctx, client.ObjectKey{Name: "my-secret", Namespace: r.operatorNamespace}, secret); err != nil {
        return ctrl.Result{}, err
    }
    // Use secret...
}
```

To:

```go
func (r *DatabaseReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // Assume secret is in database's namespace
    secret := &corev1.Secret{}
    if err := r.Get(ctx, client.ObjectKey{Name: "my-secret", Namespace: req.Namespace}, secret); err != nil {
        return ctrl.Result{}, err
    }
    // Use secret...
}
```

### - **Provide the ability to serve different versions of your managed application / driver:**
   - Your operator could deploy a database, and different namespaces could require different versions of this database. Your operator should accommodate these requests, for example, by including a `version` field in the CRD that specifies which version to deploy.

### - **Evolve your operator APIs with non-breaking changes by adding CRD versions:**
   - If you need to change the schema of your CRD (for example, adding a new optional field), you should create a new version of your CRD (e.g., v1beta2), keeping the old one (e.g., v1beta1) for backward compatibility.

### - **Keep support older CRD versions for as long as possible:**
   - Continuing with the previous example, your operator should be capable of managing resources created with both v1beta1 and v1beta2 CRDs.

### - **Do not remove CRD versions unless you release a new major version of your operator:**
   - Suppose you decide to remove the v1beta1 version of the CRD. You should plan to do this only when you're ready to release a new major version of your operator, so users are aware that this is a breaking change.

### - **If breaking changes to APIs are required, use CRD conversion webhooks if any possible:**
   - If you need to introduce a breaking change to your CRD, use a conversion webhook. This webhook could, for example, automatically convert a v1beta1 object to a v1beta2 object when the user tries to interact with it, preserving backward compatibility.

### - **Do not assume or require cluster-scoped permissions or permissions on cluster-scoped APIs:**
   - Your operator should still work with the least amount of privileges necessary. If it used to need permissions to read a certain ConfigMap in its own namespace, now it needs permissions to read that ConfigMap in any namespace. But it doesn't suddenly need permissions to read all ConfigMaps in all namespaces.



### [Timeline](https://openshift-release.apps.ci.l2s4.p1.openshiftapps.com/)
- 4.14 ( Available only in cli, Operator CR in preview for internal)
- 4.15( no dates yet when it will be available) GA but limited support
   - clusterscope operator will be picked up by the new OLM API.
   - WATCHNAMESPACE env variable, part of the operatorgroup, does not exist in v1.


