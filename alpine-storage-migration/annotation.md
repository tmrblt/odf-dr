```
oc annotate --overwrite -n openshift-cnv hco kubevirt-hyperconverged   kubevirt.kubevirt.io/jsonpatch='[ {"op": "add", "path": "/spec/configuration/developerConfiguration/featureGates/-", "value": "VolumesUpdateStrategy"}, {"op": "add", "path": "/spec/configuration/developerConfiguration/featureGates/-", "value": "VolumeMigration"} ]'
```
