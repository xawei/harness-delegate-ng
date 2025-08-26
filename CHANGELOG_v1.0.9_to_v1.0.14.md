# Harness Delegate NG Helm Chart - Version Comparison

## v1.0.9 → v1.0.14 Detailed Changelog

### Overview
This document provides a comprehensive comparison between Harness Delegate NG Helm Chart versions v1.0.9 and v1.0.14, highlighting all structural and functional changes.

---

## 📁 File Structure Changes

### ➕ **ADDED FILES** (New in v1.0.14)

#### 1. Custom Role Binding Support
- **`templates/customRoleBinding.yaml`**
  - **Purpose**: Enables custom Kubernetes role binding for non-standard permission configurations
  - **Functionality**: Creates RoleBinding when custom role is specified in `k8sPermissionsType`
  - **Condition**: Only deployed when `harness-delegate-ng.useCustomRole` helper returns "true"
  - **⚠️ Default Behavior**: **NOT CREATED** with default values (`k8sPermissionsType: "CLUSTER_ADMIN"`)

#### 2. CCM (Cloud Cost Management) Integration
- **`templates/ccm/cost-access.yaml`**
  - **Purpose**: Provides cluster-level permissions for cost visibility features
  - **Functionality**: Creates ClusterRole and ClusterRoleBinding for cost monitoring
  - **Condition**: Only deployed when `ccm.visibility: true` and `k8sPermissionsType != "CLUSTER_ADMIN"`
  - **Permissions**: Read access to pods, nodes, events, namespaces, PVs, PVCs, deployments, jobs, metrics
  - **⚠️ Default Behavior**: **NOT CREATED** with default values (`ccm.visibility: false` and `k8sPermissionsType: "CLUSTER_ADMIN"`)

#### 3. Shared Certificates Management
- **`templates/shared_certificates/certificateConfigMap.yaml`**
  - **Purpose**: Manages certificate configuration for CI/CD pipelines
  - **Functionality**: Creates ConfigMap with certificate paths and mount targets
  - **Condition**: Only created when `shared_certificates.ci_mount_targets` is defined
  - **⚠️ Default Behavior**: **NOT CREATED** with default values (`ci_mount_targets` is empty/commented)

- **`templates/shared_certificates/certificateSecret.yaml`**
  - **Purpose**: Stores custom CA bundle for self-signed certificates
  - **Functionality**: Creates Secret with base64-encoded certificate bundle
  - **Condition**: Only created when `shared_certificates.ca_bundle` is provided
  - **⚠️ Default Behavior**: **NOT CREATED** with default values (`ca_bundle` is empty/commented)

#### 4. Legacy HPA Support
- **`templates/hpaLegacy.yaml`**
  - **Purpose**: Backward compatibility for Kubernetes versions < 1.23
  - **Functionality**: Uses `autoscaling/v2beta1` API instead of `autoscaling/v2`
  - **Condition**: Only deployed when autoscaling is enabled and Kubernetes version < 1.23
  - **⚠️ Default Behavior**: **NOT CREATED** with default values (`autoscaling.enabled: false`)

> **📋 Summary of New Files with Default Values:**
> 
> **All 5 new template files will NOT be created when using default values from either v1.0.9 or v1.0.14.** This ensures backward compatibility and prevents unexpected resource creation during upgrades. Users must explicitly enable these features by modifying the appropriate values in `values.yaml`.

### 🔄 **UPDATED FILES**

#### 1. **`Chart.yaml`**
```diff
- version: 1.0.9
+ version: 1.0.14
```

#### 2. **`values.yaml`** - Major Configuration Enhancements

**Removed Fields:**
```diff
- # Account Secret
- accountSecret: ""
```

**Added Fields:**
```yaml
# 🚀 New deployment mode specification
deployMode: "KUBERNETES"  # 📄 Used in: deployment.yaml (env vars)

# 📈 Enhanced autoscaling documentation
autoscaling:
  # Edit this if you want to enable horizontal pod autoscaling
  enabled: false  # 📄 Used in: hpa.yaml, hpaLegacy.yaml
  # ... (enhanced comments)

# 🏷️ Configurable annotations (moved from hardcoded)
annotations:
  prometheus.io/scrape: "true"     # 📄 Used in: deployment.yaml (pod annotations)
  prometheus.io/port: "3460"       # 📄 Used in: deployment.yaml (pod annotations)
  prometheus.io/path: "/api/metrics" # 📄 Used in: deployment.yaml (pod annotations)

# 🔐 External secret support
existingDelegateToken: ""  # 📄 Used in: secret.yaml (conditional creation), deployment.yaml (secret reference)

# ⬆️ Enhanced upgrader configuration
upgrader:
  # ... existing fields ...
  existingUpgraderToken: ""  # 📄 Used in: upgrader/secret.yaml (conditional creation)

# 🛡️ Enhanced security context
delegateSecurityContext:
  allowPrivilegeEscalation: false  # 📄 Used in: deployment.yaml (container securityContext)
  runAsUser: 0                     # 📄 Used in: deployment.yaml (container securityContext)

# 🔒 Shared certificates configuration
shared_certificates:
  certs_path: /shared/customer-artifacts/certificates/ca.bundle  # 📄 Used in: deployment.yaml (volume mount path)
  ca_bundle: # |                    # 📄 Used in: shared_certificates/certificateSecret.yaml
    # -----BEGIN CERTIFICATE-----
    # ...
  ci_mount_targets:               # 📄 Used in: shared_certificates/certificateConfigMap.yaml
    # - /etc/ssl/certs/ca-bundle.crt
    # - /etc/ssl/certs/ca-certificates.crt

# 🌍 Custom environment variables
custom_envs:  # 📄 Used in: deployment.yaml (container env section)

# 💾 Custom volume mounts
custom_mounts:  # 📄 Used in: deployment.yaml (container volumeMounts)

# 📦 Custom volumes
custom_volumes:  # 📄 Used in: deployment.yaml (pod volumes)

# ⏱️ Deployment stability enhancement
minReadySeconds: 120  # 📄 Used in: deployment.yaml (deployment spec)

# 💰 CCM cost visibility
ccm:
  visibility: false  # 📄 Used in: ccm/cost-access.yaml (conditional creation)
```

**Updated Fields:**
```diff
- delegateDockerImage: harness/delegate:23.02.78500
+ delegateDockerImage: harness/delegate:23.08.80104
```

#### 3. **`templates/_helpers.tpl`** - New Helper Functions

**Added Helper Functions:**
```yaml
# 🔐 Custom role detection
{{- define "harness-delegate-ng.useCustomRole" -}}
  # Logic to determine if custom role is being used
{{- end }}
# 📄 Used in: customRoleBinding.yaml (conditional creation)

# 🔒 Certificate mount volume generation
{{- define "certificate_mount_volumes" -}}
  # Generates comma-separated list of certificate mount paths
{{- end }}
# 📄 Used in: shared_certificates/certificateConfigMap.yaml (CI_MOUNT_VOLUMES)

# 🔑 Token name resolution with external secret support
{{- define "harness-delegate-ng.delegateToken" -}}
  # Returns appropriate token secret name
{{- end }}
# 📄 Used in: deployment.yaml (secret reference), secret.yaml (conditional logic)

# ⬆️ Upgrader token name resolution
{{- define "harness-delegate-ng.upgraderDelegateToken" -}}
  # Returns appropriate upgrader token secret name
{{- end }}
# 📄 Used in: upgrader/cronjob.yaml (secret reference), upgrader/secret.yaml (conditional logic)
```

#### 4. **`templates/deployment.yaml`** - Significant Enhancements

**Added Configurations:**
```diff
# ⏱️ Deployment stability enhancement
+ minReadySeconds: {{ .Values.minReadySeconds }}

# 🏷️ Annotations moved from hardcoded to configurable
metadata:
  annotations:
+   {{- toYaml .Values.annotations | nindent 8 }}  # 📍 Pod-level annotations
-   prometheus.io/scrape: "true"                   # ❌ Removed hardcoded
-   prometheus.io/port: "3460"                     # ❌ Removed hardcoded
-   prometheus.io/path: "/api/metrics"             # ❌ Removed hardcoded

# 🛡️ Enhanced security context
securityContext:
- allowPrivilegeEscalation: false               # ❌ Removed hardcoded
- runAsUser: 0                                   # ❌ Removed hardcoded
+ {{- toYaml .Values.delegateSecurityContext | nindent 12 }}  # 📍 Configurable security

# 🔒 Additional environment sources for certificates
envFrom:
  # ... existing sources ...
+ - configMapRef:                                # 📍 Certificate environment variables
+     name: {{ template "harness-delegate-ng.fullname" . }}-shared-certificates
+     optional: true

# 🌍 Custom environment variables
+ {{- with .Values.custom_envs }}               # 📍 User-defined environment variables
+ env:
+   {{- toYaml . | nindent 12 }}
+ {{- end }}

# 💾 Volume mounts for certificates and custom mounts
+ volumeMounts:
+ {{- if $.Values.shared_certificates.ca_bundle }}  # 📍 Certificate bundle mount
+   - name: certvol
+     mountPath: {{ .Values.shared_certificates.certs_path }}
+     subPath: ca.bundle
+ {{- end }}
+ {{- with .Values.custom_mounts }}              # 📍 User-defined volume mounts
+   {{- toYaml . | nindent 12 }}
+ {{- end }}

# 📦 Volumes for certificates and custom volumes
+ volumes:
+ {{- if $.Values.shared_certificates.ca_bundle }}  # 📍 Certificate secret volume
+   - name: certvol
+     secret:
+       secretName: {{ template "harness-delegate-ng.fullname" . }}-addcerts
+       items:
+       - key: ca.bundle
+         path: ca.bundle
+ {{- end }}
+ {{- with .Values.custom_volumes }}             # 📍 User-defined volumes
+   {{- toYaml . | nindent 8 }}
+ {{- end }}
```

**🔑 Updated Secret Reference:**
```diff
- secretRef:                                      # ❌ Old hardcoded reference
-   name: {{ template "harness-delegate-ng.fullname" . }}
+ secretRef:                                      # ✅ New dynamic reference
+   name: {{ include "harness-delegate-ng.delegateToken" . }}  # 📍 Supports external secrets
```

#### 5. **`templates/secret.yaml`** - 🔐 Conditional Creation

**Enhanced Logic:**
```diff
+ {{- if not .Values.existingDelegateToken }}    # 📍 Only create if no external secret
apiVersion: v1
kind: Secret
# ... rest of secret definition ...
+ {{- end }}                                     # 📍 Supports external secret management
```

### 🔄 **MODIFIED FILES** (Existing files with updates)

#### 1. **`templates/hpa.yaml`** - 📈 API Version Update
- **🔄 Change**: Updated to use `autoscaling/v2` API for Kubernetes >= 1.23
- **📋 Condition**: Added version check to complement new `hpaLegacy.yaml`
- **📄 Usage**: Works in tandem with `hpaLegacy.yaml` for version compatibility

---

## 🚀 **Feature Summary**

### **Major New Features**

1. **🔐 Custom RBAC Support**
   - Ability to use custom Kubernetes roles beyond predefined options
   - Automatic detection and binding of custom roles

2. **💰 Cloud Cost Management Integration**
   - Optional CCM visibility permissions
   - Granular cost monitoring capabilities
   - Conditional deployment based on permission type

3. **🔒 Enhanced Certificate Management**
   - Support for self-signed certificates in CI/CD pipelines
   - Configurable certificate paths and mount targets
   - Automatic certificate bundle mounting

4. **🔄 Kubernetes Version Compatibility**
   - Backward compatibility for HPA on older Kubernetes versions
   - Automatic API version selection based on cluster version

5. **🔑 External Secret Integration**
   - Support for externally managed secrets (Vault, 1Password, etc.)
   - Conditional secret creation
   - Enhanced security practices

6. **⚙️ Advanced Customization**
   - Custom environment variables support
   - Custom volume mounts and volumes
   - Configurable deployment annotations
   - Enhanced security context configuration

### **Operational Improvements**

1. **🛡️ Enhanced Security**
   - Configurable security context instead of hardcoded values
   - Support for external secret management
   - Improved RBAC granularity

2. **📈 Improved Stability**
   - `minReadySeconds` configuration for safer deployments
   - Enhanced probe configurations
   - Better upgrade handling

3. **🔧 Better Maintainability**
   - Modular template structure
   - Enhanced helper functions
   - Improved documentation and examples

---

## 🔄 **Migration Guide**

### **Breaking Changes**
- **None** - Full backward compatibility maintained

### **Deprecated Fields**
- `accountSecret` - Removed (was unused)
- `securityContext.runAsRoot` - Deprecated in favor of `delegateSecurityContext`

### **Recommended Updates**
1. Update `delegateDockerImage` to use the newer image version
2. Configure `delegateSecurityContext` instead of using deprecated `securityContext`
3. Consider enabling CCM visibility if using cost management features
4. Migrate to external secrets for enhanced security

---

## 📊 **Statistics**

- **Files Added**: 5
- **Files Modified**: 6
- **Files Deleted**: 0
- **New Configuration Options**: 15+
- **New Helper Functions**: 4
- **Backward Compatibility**: 100%

---

*Generated on: $(date)*
*Chart Versions Compared: v1.0.9 → v1.0.14*