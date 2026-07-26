# legacy-geoip-databases
Legacy GeoIP databases (.dat files) for Apache, Python, Windows Server, and Unix servers

Direct Download can be accessible from https://pcsnet.ddnsgeek.com/ddl/#search=geoip

Files in Github repository vary usefull for your SCADA on Windows or UNIX server:

- GeoIP.dat.gz
- GeoIPASNum.dat.gz
- GeoIPASNumv6.dat.gz
- GeoIPCity.dat.gz
- GeoIPCityv6.dat.gz
- GeoIPISP.dat.gz
- GeoIPISPv6.dat.gz
- GeoIP-legacy.csv.gz
- GeoIPOrg.dat.gz
- GeoIPOrgv6.dat.gz
- GeoIPv6.dat.gz


Exampe if you use Apache WEB Server with ProFTPd:
```bash
sudo apt-get install -y apache2 apache2-utils apache2-dev libapache2-mod-geoip geoip-bin geoip-database proftpd openssl proftpd-basic proftpd-mod-geoip -y
 ```
NOW OVERWRITE ALL DATABASES IN DIRECTORY for UNIX BASE SERVER:
```
/usr/share/GeoIP/
```
OR FOR WINDOWS DEPEND ON YOUR INITIAL INSTALL:
```
C:\ProgramData\GeoIP or C:\GeoIP
```
apache
# Load required modules
LoadModule mod_geoip.c

# GeoIP configuration
```bash
sudo nano /etc/apache2/mods-available/geoip.conf

<IfModule mod_geoip.c>
    ########################################################################
    # GeoIP Legacy Configuration
    # PCSNET / Piramide Studio NET - Ubuntu 24.04
    ########################################################################
    GeoIPEnable On

    # === IPv4 + IPv6 GeoIP Databases ===
    GeoIPDBFile /usr/share/GeoIP/GeoIP.dat MemoryCache
    GeoIPDBFile /usr/share/GeoIP/GeoIPv6.dat Standard
    GeoIPDBFile /usr/share/GeoIP/GeoIPCity.dat Standard
    GeoIPDBFile /usr/share/GeoIP/GeoIPCityv6.dat Standard
    GeoIPDBFile /usr/share/GeoIP/GeoIPASNum.dat Standard
    GeoIPDBFile /usr/share/GeoIP/GeoIPASNumv6.dat Standard
    GeoIPDBFile /usr/share/GeoIP/GeoIPISP.dat Standard
    GeoIPDBFile /usr/share/GeoIP/GeoIPOrg.dat Standard

    # === Performance Cache Options ===
    # MemoryCache = fastest (in-memory)
    # Standard = normal file access
    # IndexCache / CheckCache = intermediate speed
    # Example for large production:
    # GeoIPDBFile /usr/share/GeoIP/GeoIPCity.dat MemoryCache CheckCache

    # === UTF-8 Support ===
    GeoIPEnableUTF8 On

    # === Proxy Handling ===
    GeoIPScanProxyHeaders On
    GeoIPUseLastXForwardedForIP On
    GeoIPUseFirstNonPrivateXForwardedForIP On

    # === Output Control ===
    #   Env   → populate environment variables only
    #   All   → send to both environment and HTTP headers
    GeoIPOutput All

    ########################################################################
    # SetEnvIf directives to export GeoIP variables
    ########################################################################

    # Country
    SetEnvIf GEOIP_COUNTRY_CODE ^(.*)$ GEOIP_COUNTRY_CODE=$1
    SetEnvIf GEOIP_COUNTRY_NAME ^(.*)$ GEOIP_COUNTRY_NAME=$1

    # Continent
    SetEnvIf GEOIP_CONTINENT_CODE ^(.*)$ GEOIP_CONTINENT_CODE=$1

    # City
    SetEnvIf GEOIP_CITY ^(.*)$ GEOIP_CITY=$1
    SetEnvIf GEOIP_REGION ^(.*)$ GEOIP_REGION=$1
    SetEnvIf GEOIP_REGION_NAME ^(.*)$ GEOIP_REGION_NAME=$1
    SetEnvIf GEOIP_POSTAL_CODE ^(.*)$ GEOIP_POSTAL_CODE=$1
    SetEnvIf GEOIP_LATITUDE ^(.*)$ GEOIP_LATITUDE=$1
    SetEnvIf GEOIP_LONGITUDE ^(.*)$ GEOIP_LONGITUDE=$1
    SetEnvIf GEOIP_DMA_CODE ^(.*)$ GEOIP_DMA_CODE=$1
    SetEnvIf GEOIP_METRO_CODE ^(.*)$ GEOIP_METRO_CODE=$1
    SetEnvIf GEOIP_AREA_CODE ^(.*)$ GEOIP_AREA_CODE=$1
    SetEnvIf GEOIP_TIME_ZONE ^(.*)$ GEOIP_TIME_ZONE=$1

    # Organization / ISP / ASN
    SetEnvIf GEOIP_ORGANIZATION ^(.*)$ GEOIP_ORGANIZATION=$1
    SetEnvIf GEOIP_ISP ^(.*)$ GEOIP_ISP=$1
    SetEnvIf GEOIP_ASNUM ^(.*)$ GEOIP_ASNUM=$1
    SetEnvIf GEOIP_ASORG ^(.*)$ GEOIP_ASORG=$1

    # IP Address (captured by mod_geoip)
    SetEnvIf GEOIP_ADDR ^(.*)$ GEOIP_ADDR=$1

    ########################################################################
    # Example usage (optional)
    ########################################################################
    # Header always set X-GeoIP-Country-Code "%{GEOIP_COUNTRY_CODE}e"
    # Header always set X-GeoIP-City "%{GEOIP_CITY}e"
    # Header always set X-GeoIP-IP "%{GEOIP_ADDR}e"
</IfModule>
 ```

## Sample for SCADA protect login only from your countires you have home, business place, ranch or weekend house:
```bash
Example access only from EU countries SCADA setup SQL Cybrotech industrial controllers extra python module GeoIP:
UPDATE `system_countries` 
SET `enabled` = CASE 
    WHEN `country_code` IN ('AT', 'BE', 'BG', 'HR', 'CY', 'CZ', 'DK', 'EE', 'FI', 'FR', 'DE', 'GR', 'HU', 'IE', 'IT', 'LV', 'LT', 'LU', 'MT', 'NL', 'PL', 'PT', 'RO', 'SK', 'SI', 'ES', 'SE', 'GB') 
    THEN '1' 
    ELSE '0' 
END;

Apache .htaccess config put this directive in SCADA root dir or your CMS for business or private use (block all countries to log in only allow EU countires):
HERE IS EXAMPLE ALL COUNTRIES CODES SO YOU ONLY DELETE FROM THIS LIST COUNTRIES YOU WANNA HAVE ACCESS ALL REDIRECT TO FIX ERROR PAGE error.html:

<IfModule mod_rewrite.c>
RewriteEngine on
RewriteCond %{REQUEST_METHOD} (POST|GET|HEAD|OPTIONS|PROPFIND|PUT|CONNECT|DEBUG|DELETE|MOVE|TRACE|TRACK)
RewriteCond %{ENV:GEOIP_COUNTRY_CODE} ^(A1|A2|O1|AD|AE|AF|AG|AI|AL|AM|AO|AP|AQ|AR|AS|AT|AU|AW|AX|AZ|BA|BB|BD|BE|BF|BG|BH|BI|BJ|BL|BM|BN|BO|BQ|BR|BS|BT|BV|BW|BY|BZ|CA|CC|CD|CF|CG|CH|CI|CK|CL|CM|CN|CO|CR|CU|CV|CW|CX|CY|CZ|DE|DJ|DK|DM|DO|DZ|EC|EE|EG|EH|ER|ES|ET|EU|FI|FJ|FK|FM|FO|FR|GA|GB|GD|GE|GF|GG|GH|GI|GL|GM|GN|GP|GQ|GR|GS|GT|GU|GW|GY|HK|HM|HN|HR|HT|HU|ID|IE|IL|IM|IN|IO|IQ|IR|IS|IT|JE|JM|JO|JP|KE|KG|KH|KI|KM|KN|KP|KR|KW|KY|KZ|LA|LB|LC|LI|LK|LR|LS|LT|LU|LV|LY|MA|MC|MD|ME|MF|MG|MH|MK|ML|MM|MN|MO|MP|MQ|MR|MS|MT|MU|MV|MW|MX|MY|MZ|NA|NC|NE|NF|NG|NI|NL|NO|NP|NR|NU|NZ|OM|PA|PE|PF|PG|PH|PK|PL|PM|PN|PR|PS|PT|PW|PY|QA|RE|RO|RS|RU|RW|SA|SB|SC|SD|SE|SG|SH|SI|SJ|SK|SL|SM|SN|SO|SR|SS|ST|SV|SX|SY|SZ|TC|TD|TF|TG|TH|TJ|TK|TL|TM|TN|TO|TR|TT|TV|TW|TZ|UA|UG|UM|US|UY|UZ|VA|VC|VE|VG|VI|VN|VU|WF|WS|YE|YT|ZA|ZM|ZW)$
RewriteRule ^(.*)$ https://%{SERVER_NAME}/error.html [R=302,L]
</IfModule>

Example shorter version allow only EU countries + GB for POST for login files change this for your login files you use:

<IfModule mod_rewrite.c>
RewriteEngine on
RewriteCond %{ENV:GEOIP_COUNTRY_CODE} !^(AT|BE|BG|HR|CY|CZ|DK|EE|FI|FR|DE|GR|HU|IE|IT|LV|LT|LU|MT|NL|PL|PT|RO|SK|SI|ES|SE|GB)$
RewriteCond %{REQUEST_URI} ^/(login\.php|signup\.php|login|signup|signin)$
RewriteCond %{REQUEST_METHOD} POST
RewriteRule ^(.*)$ https://%{SERVER_NAME}/error.html [L,R=302]
</IfModule>

Example for block all countries except Spain for method POST|HEAD|OPTIONS|PROPFIND|PUT|CONNECT|DEBUG|DELETE|MOVE|TRACE|TRACK:

<IfModule mod_rewrite.c>
RewriteEngine on
RewriteCond %{REQUEST_METHOD} (POST|HEAD|OPTIONS|PROPFIND|PUT|CONNECT|DEBUG|DELETE|MOVE|TRACE|TRACK)
RewriteCond %{ENV:GEOIP_COUNTRY_CODE} !^ES$
RewriteRule ^(.*)$ https://%{SERVER_NAME}/error.html [R=301,L]
</IfModule>

Example for SCADA or Custom software using string query or direct uri for login/signup allow only Italy - Some SCADA system have python base scripts so uri is example /admin/login or similar depend of your WEBSCADA developers - Personally I use Cybrotech but I sugest if you use WEBSCADA by China products as they are cheapest and much much better than our WESTERN ones, softwares are updated regulary and maintained so you will have all your SCADA system localhost not on XY public servers my suggestion KingSCADA 4.0 by Beijing WellinTech and FUXA + ARM Controller:

<IfModule mod_rewrite.c>
RewriteEngine on
# Block SIGNUP only for all countries except Italy (IT)
# Check if it's a POST request to the auth.php endpoint with signup option
RewriteCond %{REQUEST_METHOD} POST
RewriteCond %{QUERY_STRING} option=signup [NC]
RewriteCond %{REQUEST_URI} /endpoints/auth/signup [NC,OR]
RewriteCond %{REQUEST_URI} /webhooks/auth\.php [NC]
RewriteCond %{ENV:GEOIP_COUNTRY_CODE} !^IT$
RewriteRule ^ - [F,L]

# Optional: Return a custom error page instead of 403
# RewriteRule ^ - [R=302,L,NC] 
# RewriteCond %{ENV:GEOIP_COUNTRY_CODE} !^IT$
# RewriteRule ^(.*)$ /signup-blocked [R=302,L]
</IfModule>

Old Scada working one localhost cybrotech is accesible here:

https://pcsnet.ddnsgeek.com/ddl/#search=app

but new one is accessible on cybrotech.com and is in developing stage

https://cybrotech.com/wp-content/uploads/2024/07/CybroEdgeToolkit.zip?v=ac5585d98646

info here about old one and new one:

https://pcsnet.ddnsgeek.com/install-namestitev-web-scada-cybrotech-raspberry-pi/

and

https://pcsnet.ddnsgeek.com/forums/forum/webscada/

 ```

# SCADA Systems recomandations

🥇 Top Recommendation: KingSCADA 4.0 by Beijing WellinTech  
For a high-end, enterprise-grade SCADA system that is fully "localized" and updated, KingSCADA 4.0 is an excellent choice. It offers a modern web-based frontend and supports a wide range of controllers and inverter brands. 

Software & Updates: A commercial platform that is regularly updated by WellinTech, a major industrial software company. It's designed for large-scale industrial applications with a "low-code" drag-and-drop interface for building monitoring frontends. 

Scalability: Built with a flexible architecture to handle up to 5 million data points, supporting large systems. 

Cross-Platform: It supports deployment on Windows, Linux, and even HarmonyOS. 

Device Compatibility: It supports a wide array of Chinese PLC brands like Nan'ao Tuo, Inovance, Zhongdian Zhike, and Hollysys, making it a great match if you're considering switching to Chinese controllers.

🚀 Top Open-Source Alternative: FUXA + ARM Controller 
If you prefer a free, flexible solution that you can update and control yourself, this is a powerful option. The benefit is that you aren't locked into a single vendor.

The Setup: A combination of an industrial ARM controller (like the ARMxy BL370) with the FUXA open-source web-based SCADA software.

Frontend: The software is 100% web-based. You can build your monitoring dashboards directly in a browser using a visual editor and access them from any device, anywhere.

Regular Updates: This is a "do-it-yourself" approach, meaning you are responsible for managing updates. However, FUXA is a popular open-source project with an active community.

Device Compatibility: It natively supports common industrial protocols like Modbus RTU/TCP, OPC UA, MQTT, and Siemens S7 (which covers a massive range of devices and almost all modern inverters).

## INTEGRATE GEOIP LEGACY IN YOUR WEBSCADA TO PROTECT YOUR HOME OR BUSINESS PLACE OR RANCH OR WEEKEND HOUSE
FOR AI I SUGGEST TO TRY THIS DEEPSEEK MODELS AND FOR SCADA SERVER I SUGGEST TO USE ORANGEPI6 AS IS BETTER THAN RASPBERRYPI BUT IF YOU HAVE RANCH OR HOME/BUSINESS PLACE FOR SOLAR PANEL MOVEMENTS I SUGGEST RASPBERRYPI AS IT CONSUME MUCH LESS ELECTRICITY COMPARE TO ORANGEPI6 AND FOR SOLARASSISTANT I SUGGEST RASPBERRYPI4/5 BUT ORANGEPI3 WILL DO PERFECT JOB:

AI Models for Smart Home & SCADA Control
- Predictive Maintenance (detecting device failures)
- Anomaly Detection (intrusion/attack detection)
- Natural Language Processing (NLP) (voice control)
- Computer Vision (security cameras, face recognition)
- Energy Optimization (smart energy management)

