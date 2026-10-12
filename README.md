# Enterprise Cloud Architecture Project - Microsoft Azure

Benvenuto! Questo repository funge da **portfolio e vetrina tecnica** per il progetto finale da me realizzato durante il percorso in **Maggioli Academy** (Certificazione di riferimento: AZ-104 Microsoft Azure Administrator).

L'obiettivo del progetto è stato la progettazione, il deployment, l'automazione e la messa in sicurezza di un'infrastruttura cloud enterprise complessa per l'azienda target *Contoso Ltd.*, applicando le best practice del *Microsoft Azure Well-Architected Framework*.

> 💡 **Nota sul Laboratorio:** Il progetto è stato interamente sviluppato e validato all'interno di un ambiente di laboratorio sandbox (*GoDeploy*). Poiché l'ambiente era temporaneo e non è più attivo, questo spazio raccoglie la documentazione architetturale ufficiale.
>
> 📂 **[CLICCA QUI PER VISIONARE IL REPORT PDF COMPLETO](./cloud_project.pdf)** 

---

## 📐 Flusso Funzionale e Architettura Logica
L'infrastruttura realizzata integra i principali ambiti operativi in un unico flusso continuo:

1. **Identity & Governance (Entry Point):** Gestione e autenticazione centralizzata degli utenti.
2. **Networking (Topologia Hub-and-Spoke):** Instradamento e bilanciamento del traffico verso le subnet applicative isolate.
3. **Compute & Workload (Spoke VNet):** Erogazione dei servizi su istanze all'interno di un backend protetto.
4. **Observability & Operations (Monitoring):** Raccolta centralizzata di metriche, log diagnostici e gestione degli allarmi.

---

## 🚀 Dettagli Tecnici dell'Infrastruttura Implementata

### 1. Identity & Governance (Principio del Least Privilege)
* **Controllo Accessi:** Configurazione centralizzata delle identità su **Microsoft Entra ID** per eliminare l'uso di credenziali generiche o ad alto rischio nelle attività quotidiane.
* **Segregazione dei Ruoli (RBAC):** Implementazione di una matrice di permessi a livello di Resource Group (`rg-contoso-core`):
  * *Virtual Machine Contributor / Contributor:* Assegnato al team IT per la gestione operativa, escludendo la facoltà di modificare i permessi di accesso.
  * *Reader:* Assegnato al personale di audit per la sola visualizzazione dello stato delle risorse.

### 2. Infrastructure as Code (IaC) & Automazione
* **Provisioning Automatizzato:** Stesura di file di template dichiarativi JSON (**ARM Templates** - `template.json` e `parameters.json`) per l'allocazione automatica delle risorse base, azzerando le difformità e gli errori derivanti dalla configurazione manuale via interfaccia grafica.

### 3. Networking, Peering & Sicurezza Perimetrale
* **Segmentazione di Rete:** Creazione della Virtual Network principale (`GioeleVnet`) e suddivisione in subnet funzionali dedicate (`GioVnetbackend`, `GioVnetdatabase`) con piano di indirizzamento IP `10.20.0.0/16`.
* **Isolamento del Traffico:** Configurazione e associazione di **Network Security Groups (NSG)** direttamente alle subnet dei workload. Definizione di regole stringenti:
  * *Inbound:* Blocco totale del traffico non autorizzato e abilitazione selettiva delle sole porte HTTP (80/443).
  * *Outbound:* Restrizione del traffico in uscita per prevenire esfiltrazioni o comunicazioni non autorizzate.
* **Connettività Intersite:** Connessione privata e ad alte prestazioni tra la rete centrale di gestione (Hub VNet) e le reti applicative (Spoke VNet) tramite **VNet Peering** bi-direzionale, sfruttando la dorsale Microsoft senza esporre il traffico su Internet pubblica.

### 4. Compute, Alta Disponibilità & Gestione del Traffico
* **Compute Scalabile:** Provisioning di istanze Azure Virtual Machines basate su immagini Windows Server / Linux.
* **Scalabilità Automatica (VMSS):** Configurazione di un Virtual Machine Scale Set (`vmss1`) con regole di Autoscale basate sulle metriche di performance: *scale-out* (aggiunta di istanze) al superamento del 70% di CPU media e *scale-in* (riduzione istanze) durante le finestre temporali a basso traffico per l'ottimizzazione dei costi.
* **Traffic Management a Doppio Livello:**
  * *Layer 4 (Azure Load Balancer):* Configurazione del bilanciatore `Gioele-LB` con sonde di integrità TCP (`gioele-HealthProbe`) ogni 5 secondi per ridistribuire il traffico sulle VM del Backend Pool.
  * *Layer 7 (Application Gateway):* Implementazione di `gioele-appgw` per la gestione del traffico web HTTP/HTTPS con **Path-Based Routing**, instradando automaticamente gli utenti verso backend pool specifici in base al percorso dell'URL (es. `/image/*` o `/video/*`).

### 5. Monitoraggio, Osservabilità & Data Protection
* **Observability Centralizzata:** Raccolta, aggregazione e interrogazione dei log di connettività e diagnostica inviati dall'Agente Azure Monitor all'interno di un **Log Analytics Workspace** unificato (con verifica dei record sulla tabella `Heartbeat`).
* **Allarmi Automatici:** Impostazione di regole di Alerting (`VM Availability`) con livello di gravità Severity 3 (Informational) per intercettare cali di disponibilità.
* **Disaster Recovery (BCDR):** Configurazione di un *Recovery Services Vault* impostato con replica **Geo-ridondante (GRS)**, garantendo la replica automatica dei dati in una seconda regione geografica distante per resistere a guasti critici dell'intero data center.
* **Politiche di Backup:** Definizione della policy `gioele-backup` (pianificazione giornaliera e mantenimento snapshot per 2 giorni) con superamento del controllo preliminare d'integrità (*Backup Pre-Check: Passed*).

---
## 🏁 Risultati della Validazione
Prima dello smantellamento dell'ambiente di laboratorio, l'architettura è stata collaudata con successo verificando:
1. Il corretto isolamento iniziale delle VM e la loro raggiungibilità immediata solo dopo l'attivazione dello stato *Connected* del VNet Peering.
2. L'efficacia del Path-Based Routing tramite test di risposta HTTP da browser esterno sull'IP pubblico del Gateway, con instradamento corretto dei contenuti multimediali.
3. La corretta tracciabilità dei log prestazionali (CPU media) e l'innesco automatico dello stato *Fired* sugli avvisi al superamento delle soglie critiche.




# Enterprise Cloud Architecture Project - Microsoft Azure (ENGLISH VERSION)

Welcome! This repository serves as a **technical showcase and portfolio** for the final project I completed during my time at **Maggioli Academy** (Reference Certification: AZ-104 Microsoft Azure Administrator).

The objective of the project was the design, deployment, automation, and hardening of a complex enterprise cloud infrastructure for the target company *Contoso Ltd.*.

> 💡 **Lab Note:** The project was entirely developed and validated within a sandbox laboratory environment (*GoDeploy*). Since the environment was temporary and is no longer active, this repository hosts the official architectural documentation.
>
> 📂 **[CLICK HERE TO VIEW THE FULL PDF REPORT](./cloud_project.pdf)** 

---

## 📐 Functional Flow & Logical Architecture
The deployed infrastructure integrates key operational domains into a single, continuous flow:

1. **Identity & Governance (Entry Point):** Centralized user management and authentication.
2. **Networking (Hub-and-Spoke Topology):** Traffic routing and load balancing towards isolated application subnets.
3. **Compute & Workload (Spoke VNet):** Service delivery on compute instances within a secured backend.
4. **Observability & Operations (Monitoring):** Centralized collection of metrics, diagnostic logs, and alert management.

---

## 🚀 Technical Details of the Implemented Infrastructure

### 1. Identity & Governance (Principle of Least Privilege)
* **Access Control:** Centralized identity configuration on **Microsoft Entra ID** to eliminate the use of generic or high-risk credentials in daily operations.
* **Role-Based Access Control (RBAC):** Implementation of a permissions matrix at the Resource Group level (`rg-contoso-core`):
  * *Virtual Machine Contributor / Contributor:* Assigned to the IT team for operational management, excluding the ability to modify access permissions.
  * *Reader:* Assigned to audit personnel for read-only visibility into resource status.

### 2. Infrastructure as Code (IaC) & Automation
* **Automated Provisioning:** Stating declarative JSON configuration files (**ARM Templates** - `template.json` and `parameters.json`) for automated baseline resource allocation, eliminating discrepancies and human errors resulting from manual GUI configurations.

### 3. Networking, Peering & Perimeter Security
* **Network Segmentation:** Creation of the primary Virtual Network (`GioeleVnet`) and segmentation into dedicated functional subnets (`GioVnetbackend`, `GioVnetdatabase`) with an IP addressing scheme of `10.20.0.0/16`.
* **Traffic Isolation:** Configuration and association of **Network Security Groups (NSGs)** directly to the workload subnets. Definition of strict rules:
  * *Inbound:* Total block of unauthorized traffic and selective enablement of HTTP/HTTPS ports (80/443) only.
  * *Outbound:* Traffic restrictions to prevent unauthorized external communications or data exfiltration.
* **Intersite Connectivity:** Private, high-performance connection between the central management network (Hub VNet) and application networks (Spoke VNets) via bi-directional **VNet Peering**, leveraging the Microsoft backbone without exposing traffic to the public Internet.

### 4. Compute, High Availability & Traffic Management
* **Scalable Compute:** Provisioning of Azure Virtual Machine instances based on Windows Server / Linux images.
* **Autoscaling (VMSS):** Configuration of a Virtual Machine Scale Set (`vmss1`) with Autoscale rules based on performance metrics: *scale-out* (adding instances) when average CPU usage exceeds 70%, and *scale-in* (reducing instances) during low-traffic windows for cost optimization.
* **Dual-Layer Traffic Management:**
  * *Layer 4 (Azure Load Balancer):* Configuration of the `Gioele-LB` load balancer with TCP health probes (`gioele-HealthProbe`) every 5 seconds to redistribute traffic across the Backend Pool VMs.
  * *Layer 7 (Application Gateway):* Implementation of `gioele-appgw` to manage HTTP/HTTPS web traffic using **Path-Based Routing**, automatically routing users to specific backend pools based on the URL path (e.g., `/image/*` or `/video/*`).

### 5. Monitoring, Observability & Data Protection
* **Centralized Observability:** Aggregation, collection, and querying of connectivity and diagnostic logs sent by the Azure Monitor Agent into a unified **Log Analytics Workspace** (validated via records in the `Heartbeat` table).
* **Automated Alerts:** Setting up Alerting rules (`VM Availability`) with Severity 3 (Informational) level to intercept drops in resource availability.
* **Disaster Recovery (BCDR):** Configuration of a *Recovery Services Vault* set up with **Geo-redundant Storage (GRS)** replication, ensuring automatic data replication to a distant secondary region to withstand critical regional data center failures.
* **Backup Policies:** Definition of the `gioele-backup` policy (daily schedule with a 2-day snapshot retention) with successful validation of preliminary integrity checks (*Backup Pre-Check: Passed*).

---
## 🏁 Validation Results
Prior to the laboratory environment decommissioning, the architecture was successfully collaudated by verifying:
1. The initial isolation of the VMs and their immediate reachability only after the VNet Peering status transitioned to *Connected*.
2. The effectiveness of Path-Based Routing via external browser HTTP response tests on the Gateway's public IP, resulting in correct multimedia content routing.
3. The precise traceability of performance logs (average CPU) and the automated triggering (*Fired* status) of alerts upon exceeding critical thresholds.

