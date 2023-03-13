```mermaid
@startuml


skinparam nodesep 10
skinparam ranksep 10

' Kubernetes
!define KubernetesPuml https://raw.githubusercontent.com/dcasati/kubernetes-PlantUML/master/dist
!define ICONURL https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/v2.3.0

!includeurl KubernetesPuml/kubernetes_Common.puml
!includeurl KubernetesPuml/kubernetes_Context.puml
!includeurl KubernetesPuml/kubernetes_Simplified.puml

!includeurl KubernetesPuml/OSS/KubernetesSvc.puml
!includeurl KubernetesPuml/OSS/KubernetesPod.puml
!includeurl KubernetesPuml/OSS/KubernetesSecret.puml
!includeurl KubernetesPuml/OSS/KubernetesCm.puml
!includeurl KubernetesPuml/OSS/KubernetesJob.puml
!includeurl KubernetesPuml/OSS/KubernetesCronjob.puml
!includeurl KubernetesPuml/OSS/KubernetesIng.puml
!includeurl KubernetesPuml/OSS/KubernetesDeploy.puml


!includeurl ICONURL/common.puml
!includeurl ICONURL/font-awesome-5/cloudflare.puml
!includeurl ICONURL/devicons2/postgresql.puml
actor User
left to right direction

FA5_CLOUDFLARE(cf) #orange
package "IONOS PGDBSaaS" {
  DEV2_POSTGRESQL(gluDb, "GLU") #336791
}
' Kubernetes Components
Cluster_Boundary(cluster, "Kubernetes Cluster") {
    Namespace_Boundary(ns, "glu") {
        KubernetesDeploy(cfTunnel, "cloudflare-tunnel", "")
        KubernetesPod(cfPod1, "tunnel pod", "")
        KubernetesPod(cfPod2, "tunnel pod", "")
        KubernetesIng(ingress, "glu.bettermarks.loc", "100")
        KubernetesSvc(api, "glu-api-service", "")
        KubernetesPod(apiPod1, "glu-pod", "")
        KubernetesPod(apiPod2, "glu-pod", "")
        KubernetesSvc(static, "glu-static-service", "")
        KubernetesPod(staticPod, "glu-static-pod", "")
        KubernetesSvc(apiPacts, "glu-pacts-api-service", "")
        KubernetesPod(apiPactsPod1, "glu-pacts-pod", "")
        KubernetesSvc(pactsDb, "glu-pacts-db", "")
        KubernetesPod(pactsDBPod1, "glu-pacts-db-pod", "")

        KubernetesSecret(appSecret, "glu-secret", "")
        KubernetesSecret(pgSecret, "glu-postgres-secret", "")
        KubernetesSecret(oidcKey, "glu-oidc-rsa-private-key", "")
        KubernetesSecret(pgPactSecret, "glu-pacts-db", "")

        frame "Jobs" {
          KubernetesJob(fixtureJob, "Runs Fixtures", "")
          KubernetesJob(migrationJob, "Runs Migrations", "")
          KubernetesCronjob(refreshMV, "Materialize View Refresh cron", "")
        }


    }
}

Rel(User, cf, "glu.bettermarks.com")
Rel(cf, cfTunnel, "proxies to tunnel")
Rel(cfTunnel, cfPod1, "loadbalance traffic")
Rel(cfTunnel, cfPod2, "loadbalance traffic")
Rel(cfPod1, ingress, "routes traffic")
Rel(cfPod2, ingress, "routes traffic")

Rel(ingress,api,"/ route")
Rel(api,apiPod1,"loadbalance traffic")
Rel(api,apiPod2,"loadbalance traffic")
Rel(appSecret, api, "Application secrets")
Rel(pgSecret, api, "DB Secrets")
Rel(oidcKey, api, "OIDC Key")
Rel(apiPod1, gluDb, "r/w Data")
Rel(apiPod2, gluDb, "r/w Data")


Rel(ingress,static,"/static route", "2")
Rel(static,staticPod,"loadbalance traffic")

Rel(ingress,apiPacts,"/pact route", "2")
Rel(apiPacts,apiPactsPod1,"loadbalance traffic")
Rel(pactsDb,pactsDBPod1,"database")
Rel(apiPactsPod1,pactsDb,"loadbalance traffic")
Rel(pgPactSecret,pactsDb,"Pact DB Credentials")

@enduml


```
