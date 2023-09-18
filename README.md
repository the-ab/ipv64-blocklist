# v64 Firewall Blocklist & First Level Protection  
- https://ipv64.net/blocklist_blocker_nodes 
- https://ipv64.net/blocklist (Unterschiedlich wenn Eingeloggt)

hier sind mir bekannte Parser zusammengeführt in einem Repository, ein teil davon ist von https://github.com/ipv64net


---
## iptable_parser.sh

Parser for iptables blocklist

Depencys iptable-persistent & jq

* replace API Key and BID
* cron for hourly run

  Precheck `iptables-legacy-save | grep -F "icmp-port-unreachable"` if emtpy response (no blocked ip)

---
# v64_blocklist_mikrotik_parser.py
These Script Read die Mikrotik Firewall IP Address List and sent it to v64_Blocklist.

## 1. Install requirements

pip3 install -r requirements.txt

## 2. Edit global variables

Edit the global variables in the python file

## 3. Test it

run /usr/bin/python3 v64_blocklist_mikrotik_parser.py and see if everything is working correctly

## 4. Crontab
*/15 * * * * cd /root/v64_blocklist_mikrotik_parser && /usr/bin/python3 v64_blocklist_mikrotik_parser.py

## Mikrotik Example Rules to Detect something.....

add action=jump chain=input dst-port=22 in-interface-list=WANs jump-target=Tarpit protocol=tcp  
add action=jump chain=input dst-port=21 in-interface-list=WANs jump-target=Tarpit protocol=tcp  
add action=jump chain=input dst-port=3389 in-interface-list=WANs jump-target=Tarpit protocol=tcp  
add action=jump chain=input dst-port=3306 in-interface-list=WANs jump-target=Tarpit protocol=tcp  
add action=jump chain=input dst-port=25 in-interface-list=WANs jump-target=Tarpit protocol=tcp  
add action=jump chain=input in-interface-list=WANs jump-target=Tarpit protocol=tcp psd=21,5s,10,5  
add action=add-src-to-address-list address-list=v64_Blocklist_report address-list-timeout=2h chain=Tarpit  
add action=tarpit chain=Tarpit in-interface-list=WANs protocol=tcp  

---

# v64_blocklist_fail2ban_parser.py  ( iptables auslesen )  

# v64_blocklist_crowdsec_parser.py ( ipset auslesen )  

# v64_blocklist_crowdsec_parser_pfctl.py ( pfctl tabellen auslesen in OPNsense )  

ausführen in der shell mit "python3 ./v64_blocklist_*_parser.py"

ersetze dein API key und Blocker Node ID

- Get it From IPv64.net Website  
v64_api_token="dein-api-key"
- Blocker Node ID  
v64_blocker_node_id="deine-blocker-id"

Cron Job Beispiel

*/15 *	* * *	root    python3 /opt/v64_blocklist_crowdsec_parser.py > /dev/null  
*/15 *	* * *	root    python3 /opt/v64_blocklist_crowdsec_parser_pfctl.py > /dev/null  
*/15 *	* * *	root    python3 /opt/v64_blocklist_fail2ban_parser.py > /dev/null  

Cron Job OPNsense Weboberfläche  
Datei actions_v64-blocklist.conf Speichern in "/usr/local/opnsense/service/conf/actions.d"   
script pfad wen notwendig anpassen, anschliessend "service configd restart"  
jetzt kann der Cron Job in der Weboberfläche unter System/Einstellungen/Cron ausgewählt werden
