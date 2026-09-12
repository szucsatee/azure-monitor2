
# Azure Linux VM Monitorozás és Riasztás Beállítási Útmutató


Ebben a repositoryban az `Azure Monitor` működését mutatom be.
Szó esik Vm-ek létrehozásáról és monitorozásáról, a loggyűjtésről, riasztási rendszerek beállításáról és teszteléséről.


Mivel is készülünk:
##  Tartalomjegyzék
1. [Előfeltételek](#-előfeltételek)
2. [Linux VM-ek létrehozása](#1-linux-vm-ek-létrehozása)
3. [Azure Monitor – Metrikák beállítása](#2-azure-monitor--metrikák-beállítása)
4. [Data Collection Rule létrehozása és naplók engedélyezése](#3-data-collection-rule-létrehozása-és-naplók-engedélyezése)
5. [Logok lekérdezése KQL-lel](#4-logok-lekérdezése-kql-lel)
6. [Riasztások tesztelése és CPU terhelés generálása](#5-riasztások-tesztelése-és-cpu-terhelés-generálása)

---

Lássunk neki!

##  Előfeltételek
* **Előfizetés:** Saját Azure előfizetésedre lesz szükséged
* **Erőforráscsoport:** Hozz létre egy új erőforráscsoportot
* **Régió:** Én a `Sweden Central` -t használom
* **Virtuális gépek létrehozása:** `vm-linux01`, `vm-linux02`, később létrehozhatunk több gépet is.

---

##  Végrehajtott lépések

### 1. Linux VM-ek létrehozása

#### 1.1. Alapbeállítások (Basics tab)
* **Erőforráscsoport:** (A létrehozott erőforráscsoportod)
* **Virtuális gép neve:** `vm-linux01` *(a többi gépnél értelemszerűen módosítandó)*
* **Régió** `Sweden Central`
* **Rendelkezésre állási beállítások** `Nem szükségesinfrastruktúráls redundanccia`
* **Kép (Image):** `Ubuntu Server 24.04 LTS - x64 Gen2`
* Gép típusa: `Standard_B1s` én a költségek miatt választom ezt.
* *1 magos processzor, erre később figyelj*
* **Hitelesítés típusa:** SSH-kulcs
  * **Kulcspár neve:** `figyelo`
* **Bejövő port:** SSH (22)

#### 1.2. Lemezek (Disks tab)
* **OS lemez típusa:** Én  `Standard HDD`-t használtam a költségek miatt

#### 1.3. Figyelés vagy Monitorozás (Monitoring tab)
##### Riasztások (Alerts):
* **Ajánlott riasztási szabályok engedélyezése:** `IGEN` - pipáld ki
* **Percentage CPU > 70%:** `IGEN` (Kritikus)
* **legördülőben** válaszd ki z étresítés `súlyosság`-át
* **Available Memory Bytes < 0.25GB (250 MB):** `IGEN` (Hiba)
* **Email:** Automatikusan kitöltve

<img width="1906" height="884" alt="image" src="https://github.com/user-attachments/assets/f7ca412e-aea0-43f0-b03f-0faab1f4536b" />




##### Diagnosztika & Üzemállapot:
* **Rendszerindítási diagnosztika:** Engedélyezés a felügyelt tárfiókkal (ajánlott)
* **Vendég operációs rendszer diagnosztikájának engedélyezése:** `NEM`
* **Alkalmazás állapotfigyelésének engedélyezése:** `NEM`

#### 1.4. Utolsó lépés - Létrehozás
1. Kattints a **"Felülvizsgálat + létrehozás"** gombra.
2. Ellenőrizd a beállításokat.
3. Kattints a **"Létrehozás"** gombra.
4. Az *SSH kulcs letöltése* ablakban válaszd a **"Privát kulcs letöltése és erőforrás létrehozása"** opciót.
5. Mentsd a kulcsot biztonságos helyre. A gép pár perc múlva elérhető lesz.

---

### 2. Azure Monitor – Metrikák beállítása

#### 2.1. Azure Monitor megnyitása
* Menj az **Azure Portal** → **Monitor** felületre.
* Válaszd a bal oldali menüben a **Metrikák** lehetőséget.

#### 2.2. Hatókör kiválasztása
1. Kattints a **"Válasszon hatókört"** gombra.
2. Pipáld be az alábbiakat:
   * **Erőforráscsoport:** (A korábban létrehozott erőforráscsoportod.)
   * **Erőforrástípus:** `Virtuális gép`
   * **Erőforrás:** `vm-linux01` *(a többi VM is hozzáadható később)*
3. Kattints az **Alkalmaz** gombra.

#### 2.3. CPU metrika hozzáadása
* **Metrika:** `Percentage CPU`
* **Összesítés:** `Average` (átlag)
<img width="1897" height="747" alt="image" src="https://github.com/user-attachments/assets/66036fea-d2e5-4682-a7b8-0e1b4f3270fc" />


#### 2.4. Memória metrika hozzáadása
1. Kattints a **"+ Metrika hozzáadása"** opcióra.
2. **Metrika:** `Available Memory Percentage`
3. **Összesítés:** `Average` (átlag)

#### 2.5. Dashboard létrehozása és mentése
1. Kattints a **Rögzítés az irányítópulton** gombra.
2. Válaszd az **Új megosztott irányítópult** lehetőséget.
3. **Irányítópult neve:** `Szerverek`
4. Kattints a **Mentés** gombra.
5. Itt érjük el az irányítópultot:

   <img width="1703" height="891" alt="image" src="https://github.com/user-attachments/assets/43af0f7a-808c-4f7e-ad6f-0aac737ad14d" />


---

### 3. Log Analytics Munkaterületek (Workspaces) és Data Collection Rules (Adatgyűjtési szabály0ok) létrehozása és naplók engedélyezése

#### A Log Analytics workspace(munkaterületek) és Data collection Rules (Adatgyűjtési szabályok beállításában nagy segítségemre volt ez:
[VM naplózás lelke: Log Analytics és Data Collection Rules](https://cloudmentor.hu/vm-naplozas-lelke-log-analytics-es-data-collection-rules/)*


#### 3.1. Naplók engedélyezése
* **Azure Portal** → **Virtuális gépek** → `vm-linux01` *(a többi VM-nél is elvégezhető)*
* Bal oldali menü → **Figyelés** → **Naplók**
* Kattints az **Engedélyezés** gombra.

#### 3.2. Data Collection Rule beállítása
* **Szabály neve legyen példáu:** `vm-logs`
* **Előfizetés:** Saját előfizetésed.
* **Erőforráscsoport:** (Ismét a korábban létrehozott erőforráscsoportod)
* **Régió:** `Sweden Central`
* **Log Analytics workspace:** `default` (automatikusan kiválasztva)
* Kattints a **Felülvizsgálat + létrehozás** → **Létrehozás** gombra.

>  **FONTOS:** Várj 25-35 percet, mire a logok megjelennek a rendszerben!

---

### 4. Logok lekérdezése KQL-lel

A lekérdezések futtatásához menj ide: **Azure Portal** → **Monitor** → **Naplók**, majd állítsd be a hatókört a kívánt gépekre, vm-ekre. (pl. `vm-linux01`).

#### 4.1. Alapvető Heartbeat ellenőrzés
```kql
Heartbeat
| where Computer contains "vm-linux"
| take 10
```
*Kattints a **Futtatás** gombra. Ha az adatok megjelennek, a loggyűjtés sikeresen működik, valahogy így.!*

<img width="1694" height="898" alt="image" src="https://github.com/user-attachments/assets/5d0c1762-61fc-4a94-9348-4b164da46697" />


#### 4.2. Néhány hasznos KQL lekérdezés:

 **CPU metrikák (időbeli változás):**
```kql
InsightsMetrics
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AvgCPU = avg(Val) by bin(TimeGenerated, 5m)
| render timechart
```

 **Memória használat (időbeli változás):**
```kql
InsightsMetrics
| where Namespace == "Memory" and Name == "AvailableMB"
| summarize AvgMemory = avg(Val) by bin(TimeGenerated, 5m)
| render timechart
```
<img width="1878" height="834" alt="image" src="https://github.com/user-attachments/assets/463165de-8844-4a37-9436-2009d9442c9a" />


 **Hálózati forgalom:**
```kql
InsightsMetrics
| where Namespace == "Network"
| where Name in ("ReadBytesPerSecond", "WriteBytesPerSecond")
| summarize NetworkMBps = sum(Val)/1024/1024 by bin(TimeGenerated, 5m), Name
| render timechart
```

 **VM állapot ellenőrzése:**
```kql
Heartbeat
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| extend Status = iff(LastHeartbeat > ago(5m), "Online", "Offline")
| project Computer, LastHeartbeat, Status
```

---

### 5. Riasztások tesztelése és CPU terhelés generálása

#### 5.1. Linux VM CPU terhelés (`vm-linux01` - `04`)
Csatlakozz a VM-hez **SSH**-n keresztül, majd futtasd a következő parancsokat, én a VM-eket az Azureben nyitottam meg a Soros Konzol lehetőséggel.


# Csomaglista frissítése és a stress-ng telepítése
```bash
sudo apt update
```

```bash
sudo apt install -y stress-ng
```


# CPU terhelés indítása 2 magon, 5 percig (300 másodperc)

A VM-eket az Azureben nyitottam meg a Soros Konzol lehetőséggel.

```bash
stress-ng --cpu 2 --timeout 300s
```
*cpu 2 --2 magot terhel, ezzel erősebb, vagy gyengébb gép esetén tudsz módosítani, ugyanígy a 300s ami lehet 5m is akár vagy több és kevesebb is.*
*A riasztás ~5-10 perc alatt aktiválódik, és a beállított e-mail címre értesítés érkezik.*

<img width="1731" height="891" alt="image" src="https://github.com/user-attachments/assets/1d53e66f-befd-469e-8c19-d59101c221cc" />


Ha a folyamatot idő előtt le szeretnéd állítani:
```bash
sudo pkill stress-ng
```

#### 5.2. Windows VM CPU terhelés (`vm-win01`)
Csatlakozz a géphez **RDP**-n keresztül, majd nyiss egy **PowerShell** ablakot, ém a VM-eket az Azureben nyitottam meg a Soros Konzol lehetőséggel.

**Futtatás:**
```powershell
for (i=0; i -lt 4; i++) Start-Job while (true) {} } }
```

**Leállítás:**
```powershell
Get-Job | Stop-Job
```
*javaslom, hogy használj CTRL+C és CTRL+V billenytyűkombinációt és várj pár másodpercet*
*a terhelés alatt nagyon belassul a rendszer*

#### 5.3. Riasztás ellenőrzése az Azure felületén
* Menj ide: **Azure Portal** → **Monitor** → **Riasztások**
* Várj 5-10 percet.
* Ellenőrizd a postafiókodat az e-mailért.
  <img width="1436" height="352" alt="image" src="https://github.com/user-attachments/assets/ee47e0c1-75da-49c3-bcfb-9d8bc144accd" />

* A riasztásnak meg kell jelennie az **"Aktivált riasztások"** listájában is, valahogy így:
  <img width="1877" height="819" alt="image" src="https://github.com/user-attachments/assets/f12c1adb-245c-4055-aa58-21677996edf9" />


*Forrás: Kiss Tibor - cloudmentor.hu - köszönöm Tibornak
