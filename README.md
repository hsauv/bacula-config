# bacula-config
Director {
  Name = bacula-dir
  DIRport = 9101
  QueryFile = "/opt/bacula/scripts/query.sql"
  WorkingDirectory = "/opt/bacula/working"
  PidDirectory = "/opt/bacula/working"
  Maximum Concurrent Jobs = 20
  Password = "director-password"
  Messages = Daemon
}

The provided configuration block is for Bacula’s Director, which is the central component responsible for managing backup and restore operations. 
While the configuration is straightforward, adapting it to a Kubernetes or custom environment involves several considerations to ensure reliability, scalability, and proper integration.

Adapting to Kubernetes or Custom Environments

1. Use PersistentVolumes for Stateful Data
Mount volumes for directories like WorkingDirectory and PidDirectory to ensure state persistence across restarts:
volumeMounts:
  - name: bacula-working-dir
    mountPath: /opt/bacula/working
volumes:
  - name: bacula-working-dir
    persistentVolumeClaim:
      claimName: bacula-working-pvc

2. Store Sensitive Information in Secrets
Avoid hardcoding sensitive information like passwords in configuration files. Use Kubernetes secrets:
env:
  - name: BACULA_PASSWORD
    valueFrom:
      secretKeyRef:
        name: bacula-secret
        key: director-password

Update the configuration to reference the environment variable:
Password = "${BACULA_PASSWORD}"

3. Ensure ConfigMap for Configuration
Use a ConfigMap to manage Bacula configuration files:
apiVersion: v1
kind: ConfigMap
metadata:
  name: bacula-dir-config
data:
  bacula-dir.conf: |
    Director {
      Name = bacula-dir
      DIRport = 9101
      QueryFile = "/opt/bacula/scripts/query.sql"
      WorkingDirectory = "/opt/bacula/working"
      PidDirectory = "/opt/bacula/working"
      Maximum Concurrent Jobs = 20
      Password = "${BACULA_PASSWORD}"
      Messages = Daemon
    }

4. Network Configuration
Ensure proper networking in Kubernetes. Use a Service to expose the Director port (9101):

apiVersion: v1
kind: Service
metadata:
  name: bacula-dir
spec:
  selector:
    app: bacula-dir
  ports:
    - protocol: TCP
      port: 9101
      targetPort: 9101
      
5. Liveness and Readiness Probes
Add probes to ensure the Director is healthy:
livenessProbe:
  tcpSocket:
    port: 9101
  initialDelaySeconds: 30
  periodSeconds: 10
readinessProbe:
  tcpSocket:
    port: 9101
  initialDelaySeconds: 10
  periodSeconds: 5
