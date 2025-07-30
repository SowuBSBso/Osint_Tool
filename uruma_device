from Config.Util import *
from Config.Config import *
from datetime import datetime

import sys
import os
import contextlib
import random
import re
import requests
import json
import dns.resolver
import phonenumbers
from phonenumbers import geocoder, carrier, timezone
import concurrent.futures
import threading
import subprocess
import socket

# --- Modules optionnels, essayer d'importer et gérer erreurs ---
try:
    import instaloader
except ImportError:
    instaloader = None

# --- Variables globales utiles ---
number_valid = 0
number_invalid = 0
lock = threading.Lock()

# --- Helpers généraux ---
def clear_console():
    os.system('cls' if os.name == 'nt' else 'clear')

def pause():
    input(f"\n{BEFORE + current_time_hour() + AFTER} {INPUT} Press Enter to continue...{reset}")

def error_and_continue(e):
    print(f"{BEFORE + current_time_hour() + AFTER} {ERROR} {str(e)}")
    pause()

# --- Fonctionnalités ---

def instagram_lookup():
    if instaloader is None:
        print(f"{BEFORE + current_time_hour() + AFTER} {ERROR} Module instaloader is not installed.")
        pause()
        return

    Title("Instagram Account Lookup")

    def Search(username):
        @contextlib.contextmanager
        def Output():
            with open(os.devnull, 'w') as devnull:
                old_stdout = sys.stdout
                old_stderr = sys.stderr
                sys.stdout = devnull
                sys.stderr = devnull
                try:
                    yield
                finally:
                    sys.stdout = old_stdout
                    sys.stderr = old_stderr

        with Output():
            loader = instaloader.Instaloader()
            profile = instaloader.Profile.from_username(loader.context, username)

        return loader, profile

    try:
        username = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Instagram Username -> {reset}")
        loader, profile = Search(username)
    except Exception:
        print(f"{BEFORE + current_time_hour() + AFTER} {ERROR} You have exceeded your limit or the user does not exist, try again later.")
        pause()
        return

    Slow(f"""    
{white}────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 {INFO_ADD} Full Name       : {white}{profile.full_name}
 {INFO_ADD} Username        : {white}{profile.username}
 {INFO_ADD} Instagram Id    : {white}{profile.userid}
 {INFO_ADD} Biography       : {white}{profile.biography}
 {INFO_ADD} Profile Url     : {white}https://instagram.com/{profile.username}
 {INFO_ADD} Profile Photo   : {white}{profile.profile_pic_url}
 {INFO_ADD} Publications    : {white}{profile.mediacount}
 {INFO_ADD} Subscribers     : {white}{profile.followers}
 {INFO_ADD} Subscriptions   : {white}{profile.followees}
 {INFO_ADD} Verified        : {white}{'True' if profile.is_verified else 'False'}
 {INFO_ADD} Private Account : {white}{'True' if profile.is_private else 'False'}
 {INFO_ADD} Pro Account     : {white}{'True' if profile.is_business_account else 'False'}""")

    if profile.is_business_account:
        print(f"    {INFO_ADD} Category Pro    : {white}{profile.business_category_name}")

    print(f"{white}────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────")

    if not profile.is_private or loader.context.username == profile.username:
        try:
            posts = profile.get_posts()
            for i, post in enumerate(posts):
                Slow(f"""    
 {INFO_ADD} Publication n°{i+1}
 {INFO_ADD} URL        : {white}https://www.instagram.com/p/{post.shortcode}/
 {INFO_ADD} Date       : {white}{post.date}
 {INFO_ADD} Likes      : {white}{post.likes}
 {INFO_ADD} Comments   : {white}{post.comments}
 {INFO_ADD} Legend     : {white}{post.caption}
{white}────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────""")
                if i == 4:
                    break
            print()
        except Exception:
            print(f"\n{BEFORE + current_time_hour() + AFTER} {ERROR} Error retrieving posts.")

    pause()


def email_lookup():
    Title("Email Lookup")

    def GetEmailInfo(email):
        info = {}
        try: domain_all = email.split('@')[-1]
        except: domain_all = None

        try: name = email.split('@')[0]
        except: name = None

        try: domain = re.search(r"@([^@.]+)\.", email).group(1)
        except: domain = None

        try: tld = f".{email.split('.')[-1]}"
        except: tld = None

        try:
            mx_records = dns.resolver.resolve(domain_all, 'MX')
            mx_servers = [str(record.exchange) for record in mx_records]
            info["mx_servers"] = mx_servers
        except (dns.resolver.NoAnswer, dns.resolver.NXDOMAIN):
            info["mx_servers"] = None

        try:
            spf_records = dns.resolver.resolve(domain_all, 'SPF')
            info["spf_records"] = [str(record) for record in spf_records]
        except (dns.resolver.NoAnswer, dns.resolver.NXDOMAIN):
            info["spf_records"] = None

        try:
            dmarc_records = dns.resolver.resolve(f'_dmarc.{domain_all}', 'TXT')
            info["dmarc_records"] = [str(record) for record in dmarc_records]
        except (dns.resolver.NoAnswer, dns.resolver.NXDOMAIN):
            info["dmarc_records"] = None

        if info.get("mx_servers"):
            for server in info["mx_servers"]:
                if "google.com" in server:
                    info["google_workspace"] = True
                elif "outlook.com" in server:
                    info["microsoft_365"] = True

        return info, domain_all, domain, tld, name

    try:
        email = input(f"\n{BEFORE + current_time_hour() + AFTER} {INPUT} Email -> {reset}")
        Censored(email)
        print(f"{BEFORE + current_time_hour() + AFTER} {WAIT} Information Recovery..{reset}")
        info, domain_all, domain, tld, name = GetEmailInfo(email)

        mx_servers = ' / '.join(info["mx_servers"]) if info.get("mx_servers") else None
        spf_records = info.get("spf_records")
        dmarc_records = ' / '.join(info["dmarc_records"]) if info.get("dmarc_records") else None
        google_workspace = info.get("google_workspace")
        mailgun_validation = info.get("mailgun_validation")

        Slow(f"""
{white}────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 {INFO_ADD} Email      : {white}{email}{red}
 {INFO_ADD} Name       : {white}{name}{red}
 {INFO_ADD} Domain     : {white}{domain}{red}
 {INFO_ADD} Tld        : {white}{tld}{red}
 {INFO_ADD} Domain All : {white}{domain_all}{red}
 {INFO_ADD} Servers    : {white}{mx_servers}{red}
 {INFO_ADD} Spf        : {white}{spf_records}{red}
 {INFO_ADD} Dmarc      : {white}{dmarc_records}{red}
 {INFO_ADD} Workspace  : {white}{google_workspace}{red}
 {INFO_ADD} Mailgun    : {white}{mailgun_validation}{red}
{white}────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
""")
    except Exception as e:
        error_and_continue(e)

    pause()

def dox_create():
    Title("Dox Create")
    try:
        def NumberInfo(phone_number):
            try:
                parsed_number = phonenumbers.parse(phone_number, None)
                operator_phone = carrier.name_for_number(parsed_number, "fr")
                type_number_phone = "Mobile" if phonenumbers.number_type(parsed_number) == phonenumbers.PhoneNumberType.MOBILE else "Fixe"
                country_phone = phonenumbers.region_code_for_number(parsed_number)
                region_phone = geocoder.description_for_number(parsed_number, "fr")
                timezones = timezone.time_zones_for_number(parsed_number)
                timezone_phone = timezones[0] if timezones else None
            except Exception as e:
                operator_phone = "None"
                type_number_phone = "None"
                country_phone = "None"
                region_phone = "None"
                timezone_phone = "None"

            return operator_phone, type_number_phone, country_phone, region_phone, timezone_phone

        def IpInfo(ip):
            try:
                response = requests.get(f"https://{website}/api/ip/ip={ip}")
                api = response.json()
            except Exception as e:
                api = {}

            isp_ip = api.get("isp", "None")
            org_ip = api.get("org", "None")
            as_ip = api.get("as", "None")

            return isp_ip, org_ip, as_ip

        def TokenInfo(token):
            try:
                from datetime import datetime, timezone
                user = requests.get('https://discord.com/api/v8/users/@me', headers={'Authorization': token}).json()
            except Exception as e:
                user = {}

            username_discord = f"{user.get('username', 'None')}#{user.get('discriminator', 'None')}"
            display_name_discord = user.get('global_name', 'None')
            user_id_discord = user.get('id', 'None')

            try:
                avatar_url_discord = f"https://cdn.discordapp.com/avatars/{user_id_discord}/{user['avatar']}.gif"
                r = requests.get(avatar_url_discord)
                if r.status_code != 200:
                    avatar_url_discord = f"https://cdn.discordapp.com/avatars/{user_id_discord}/{user['avatar']}.png"
            except Exception as e:
                avatar_url_discord = "None"

            try:
                created_at_discord = datetime.fromtimestamp(((int(user_id_discord) >> 22) + 1420070400000) / 1000, timezone.utc)
            except Exception as e:
                created_at_discord = "None"

            email_discord = user.get('email', 'None')
            phone_discord = user.get('phone', 'None')

            # Simplification pour friends et gift codes (à améliorer si besoin)
            friends_discord = "None"
            gift_codes_discord = "None"

            mfa_discord = user.get('mfa_enabled', 'None')

            premium_type = user.get('premium_type', 0)
            nitro_map = {0: 'False', 1: 'Nitro Classic', 2: 'Nitro Boosts', 3: 'Nitro Basic'}
            nitro_discord = nitro_map.get(premium_type, 'False')

            return username_discord, display_name_discord, user_id_discord, avatar_url_discord, created_at_discord, email_discord, phone_discord, nitro_discord, friends_discord, gift_codes_discord, mfa_discord


        by =      input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Doxed By      : {reset}")
        reason =  input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Reason        : {reset}")
        pseudo1 = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} First Pseudo  : {reset}")
        pseudo2 = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Second Pseudo : {reset}")

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Discord Information:")
        token_input = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Token ? (y/n) -> {reset}")
        if token_input.lower() in ["y", "yes"]:
            token = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Token: {reset}")
            username_discord, display_name_discord, user_id_discord, avatar_url_discord, created_at_discord, email_discord, phone_discord, nitro_discord, friends_discord, gift_codes_discord, mfa_discord = TokenInfo(token)
        else:
            token = "None"
            username_discord =     input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Username      : {reset}")
            display_name_discord = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Display Name  : {reset}")
            user_id_discord =      input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Id            : {reset}")
            avatar_url_discord =   input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Avatar        : {reset}")
            created_at_discord =   input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Created At    : {reset}")
            email_discord =        input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Email         : {reset}")
            phone_discord =        input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Phone         : {reset}")
            nitro_discord =        input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Nitro         : {reset}")
            friends_discord =      input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Friends       : {reset}")
            gift_codes_discord =   input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Gift Code     : {reset}")
            mfa_discord =          input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Mfa           : {reset}")

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Ip Information:")
        ip_public = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Ip Publique   : {reset}")
        ip_local =  input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Ip Local      : {reset}")
        ipv6 =      input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Ipv6          : {reset}")
        vpn_pc =    input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} VPN           : {reset}")
        isp_ip, org_ip, as_ip = IpInfo(ip_public)

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Pc Information:")
        name_pc =         input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Name          : {reset}")
        username_pcc =    input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Username      : {reset}")
        displayname_pc =  input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Display Name  : {reset}")
        platform_pc =     input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Platefrom     : {reset}")
        exploitation_pc = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Exploitation  : {reset}")
        windowskey_pc =   input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Windows Key   : {reset}")
        mac_pc =          input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} MAC Adress    : {reset}")
        hwid_pc =         input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} HWID Adress   : {reset}")
        cpu_pc =          input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} CPU           : {reset}")
        gpu_pc =          input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} GPU           : {reset}")
        ram_pc =          input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} RAM           : {reset}")
        disk_pc =         input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Disk          : {reset}")
        mainscreen_pc =   input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Screen Main   : {reset}")
        secscreen_pc =    input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Screen Sec    : {reset}")

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Number Information:")
        phone_number = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Phone Number  : {reset}")
        brand_phone = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Brand         : {reset}")
        operator_phone, type_number_phone, country_phone, region_phone, timezone_phone = NumberInfo(phone_number)

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Personal Information:")
        gender =     input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Gender        : {reset}")
        last_name =  input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Last Name     : {reset}")
        first_name = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} First Name    : {reset}")
        age =        input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Age           : {reset}")
        mother =     input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Mother        : {reset}")
        father =     input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Father        : {reset}")
        brother =    input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Brother       : {reset}")
        sister =     input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Sister        : {reset}")

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Loc Information:")
        continent =   input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Continent     : {reset}")
        country =     input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Country       : {reset}")
        region =      input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Region        : {reset}")
        postal_code = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Postal Code   : {reset}")
        city =        input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} City          : {reset}")
        street =      input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Street        : {reset}")
        street_nb =   input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Street Number : {reset}")
        latitude =    input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Latitude      : {reset}")
        longitude =   input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Longitude     : {reset}")

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Bank Information:")
        credit_card = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Credit Card   : {reset}")
        bank_card =   input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Bank Card     : {reset}")
        iban =        input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} IBAN          : {reset}")

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Bank Location:")
        bank_country = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Bank Country  : {reset}")
        bank_name =    input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Bank Name     : {reset}")

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Socials:")
        facebook = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Facebook      : {reset}")
        twitter =  input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Twitter       : {reset}")
        snapchat = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Snapchat      : {reset}")
        tiktok =   input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Tiktok        : {reset}")
        linkedin = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Linkedin      : {reset}")

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Other Information:")
        ip_local_pc = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Ip Local PC   : {reset}")
        password_pc = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Password      : {reset}")
        login_pc =    input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Login         : {reset}")
        mac_pc2 =     input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Mac PC       : {reset}")
        ssn_pc =      input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} SSN           : {reset}")

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO}{yellow} Social Security Number:")
        ssn_number = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} SSN Number    : {reset}")

        # Sauvegarde dans un fichier dans dossier Dox
        if not os.path.exists("Dox"):
            os.makedirs("Dox")

        filename = f"Dox/dox_{datetime.now().strftime('%Y%m%d_%H%M%S')}.txt"

        content = f"""
Doxed By      : {by}
Reason        : {reason}
First Pseudo  : {pseudo1}
Second Pseudo : {pseudo2}

Discord Information:
Token         : {token}
Username      : {username_discord}
Display Name  : {display_name_discord}
Id            : {user_id_discord}
Avatar        : {avatar_url_discord}
Created At    : {created_at_discord}
Email         : {email_discord}
Phone         : {phone_discord}
Nitro         : {nitro_discord}
Friends       : {friends_discord}
Gift Codes    : {gift_codes_discord}
MFA           : {mfa_discord}

Ip Information:
Ip Publique   : {ip_public}
Ip Local      : {ip_local}
Ipv6          : {ipv6}
VPN           : {vpn_pc}
ISP           : {isp_ip}
Org           : {org_ip}
AS            : {as_ip}

Pc Information:
Name          : {name_pc}
Username      : {username_pcc}
Display Name  : {displayname_pc}
Platform      : {platform_pc}
Exploitation  : {exploitation_pc}
Windows Key   : {windowskey_pc}
MAC Address   : {mac_pc}
HWID Address  : {hwid_pc}
CPU           : {cpu_pc}
GPU           : {gpu_pc}
RAM           : {ram_pc}
Disk          : {disk_pc}
Screen Main   : {mainscreen_pc}
Screen Sec    : {secscreen_pc}

Number Information:
Phone Number  : {phone_number}
Brand         : {brand_phone}
Operator      : {operator_phone}
Type Number   : {type_number_phone}
Country       : {country_phone}
Region        : {region_phone}
Timezone      : {timezone_phone}

Personal Information:
Gender        : {gender}
Last Name     : {last_name}
First Name    : {first_name}
Age           : {age}
Mother        : {mother}
Father        : {father}
Brother       : {brother}
Sister        : {sister}

Loc Information:
Continent     : {continent}
Country       : {country}
Region        : {region}
Postal Code   : {postal_code}
City          : {city}
Street        : {street}
Street Number : {street_nb}
Latitude      : {latitude}
Longitude     : {longitude}

Bank Information:
Credit Card   : {credit_card}
Bank Card     : {bank_card}
IBAN          : {iban}

Bank Location:
Bank Country  : {bank_country}
Bank Name     : {bank_name}

Socials:
Facebook      : {facebook}
Twitter       : {twitter}
Snapchat      : {snapchat}
Tiktok        : {tiktok}
Linkedin      : {linkedin}

Other Information:
Ip Local PC   : {ip_local_pc}
Password      : {password_pc}
Login         : {login_pc}
Mac PC       : {mac_pc2}
SSN           : {ssn_pc}

Social Security Number:
SSN Number    : {ssn_number}
"""

        with open(filename, "w", encoding="utf-8") as f:
            f.write(content)

        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO} Informations sauvegardées dans {filename}")

        pause()

    except Exception as e:
        print(f"\n{BEFORE + current_time_hour() + AFTER} {ERROR} {e}")
        pause()

def ip_generator():
    Title("Ip Generator")

    global number_valid, number_invalid
    number_valid = 0
    number_invalid = 0
    lock = threading.Lock()

    webhook = input(f"\n{BEFORE + current_time_hour() + AFTER} {INPUT} Webhook ? (y/n) -> {reset}").lower()
    if webhook in ['y', 'yes']:
        webhook_url = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Webhook URL -> {reset}")
        CheckWebhook(webhook_url)

    try:
        threads_number = int(input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Threads Number -> {reset}"))
        if threads_number < 1:
            raise ValueError
    except ValueError:
        ErrorNumber()
        pause()
        return

    def SendWebhook(embed_content):
        payload = {
            'embeds': [embed_content],
            'username': username_webhook,
            'avatar_url': avatar_webhook
        }
        headers = {'Content-Type': 'application/json'}
        try:
            requests.post(webhook_url, data=json.dumps(payload), headers=headers, timeout=5)
        except requests.RequestException as e:
            print(f"{BEFORE + current_time_hour() + AFTER} Error sending webhook: {e}{reset}")

    def IpCheck():
        global number_valid, number_invalid
        ip = ".".join(str(random.randint(1, 255)) for _ in range(4))

        try:
            if sys.platform.startswith("win"):
                result = subprocess.run(['ping', '-n', '1', ip], capture_output=True, text=True, timeout=1)
            elif sys.platform.startswith("linux") or sys.platform.startswith("darwin"):
                result = subprocess.run(['ping', '-c', '1', '-W', '1', ip], capture_output=True, text=True, timeout=1)
            else:
                ErrorPlateform()
                return

            with lock:
                if result.returncode == 0:
                    number_valid += 1
                    print(f"{BEFORE_GREEN + current_time_hour() + AFTER_GREEN} {GEN_VALID} Logs: {white}{number_invalid} invalid - {number_valid} valid{green} Status: {white}Valid{green}  Ip: {white}{ip}{green}")
                    if webhook == 'y':
                        embed_content = {
                            'title': 'IP Valid !',
                            'description': f"**IP:**\n```{ip}```",
                            'color': color_webhook,
                            'footer': {
                                "text": username_webhook,
                                "icon_url": avatar_webhook,
                            }
                        }
                        SendWebhook(embed_content)
                else:
                    number_invalid += 1
                    print(f"{BEFORE + current_time_hour() + AFTER} {GEN_INVALID} Logs: {white}{number_invalid} invalid - {number_valid} valid{red} Status: {white}Invalid{red} Ip: {white}{ip}{red}")
            Title(f"Ip Generator - Invalid: {number_invalid} - Valid: {number_valid}")

        except Exception:
            with lock:
                number_invalid += 1
            print(f"{BEFORE + current_time_hour() + AFTER} {GEN_INVALID} Logs: {white}{number_invalid} invalid - {number_valid} valid{red} Status: {white}Invalid{red} Ip: {white}{ip}{red}")

    def Request():
        with concurrent.futures.ThreadPoolExecutor(max_workers=threads_number) as executor:
            executor.map(lambda _: IpCheck(), range(threads_number))

    try:
        while True:
            Request()
    except KeyboardInterrupt:
        print(f"\n{BEFORE + current_time_hour() + AFTER} {INFO} Interrupted by user. Exiting.")
        pause()
        return

def ip_lookup():
    Title("Ip Lookup")
    try:
        Slow(map_banner)
        ip = input(f"\n{BEFORE + current_time_hour() + AFTER} {INPUT} Ip -> {reset}")
        print(f"{BEFORE + current_time_hour() + AFTER} {WAIT} Search for information..")

        response = requests.get(f"http://ip-api.com/json/{ip}")
        api = response.json()

        status = "Valid" if api.get('status') == "success" else "Invalid"
        country = api.get('country', "None")
        country_code = api.get('countryCode', "None")
        region = api.get('regionName', "None")
        region_code = api.get('region', "None")
        zip = api.get('zip', "None")
        city = api.get('city', "None")
        latitude = api.get('lat', "None")
        longitude = api.get('lon', "None")
        timezone = api.get('timezone', "None")
        isp = api.get('isp', "None")
        org = api.get('org', "None")
        as_host = api.get('as', "None")

        Slow(f"""
{white}────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 {INFO_ADD} Status    : {white}{status}{red}
 {INFO_ADD} Country   : {white}{country} ({country_code}){red}
 {INFO_ADD} Region    : {white}{region} ({region_code}){red}
 {INFO_ADD} Zip       : {white}{zip}{red}
 {INFO_ADD} City      : {white}{city}{red}
 {INFO_ADD} Latitude  : {white}{latitude}{red}
 {INFO_ADD} Longitude : {white}{longitude}{red}
 {INFO_ADD} Timezone  : {white}{timezone}{red}
 {INFO_ADD} Isp       : {white}{isp}{red}
 {INFO_ADD} Org       : {white}{org}{red}
 {INFO_ADD} As        : {white}{as_host}{red}{reset}
{white}────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
""")

        Continue()
        Reset()
    except Exception as e:
        Error(e)

def ip_port_scanner():
    Title("Ip Port Scanner")

    try:
        def PortScanner(ip, max_threads=200):
            port_protocol_map = {
                21: "FTP", 22: "SSH", 23: "Telnet", 25: "SMTP", 53: "DNS", 69: "TFTP",
                80: "HTTP", 110: "POP3", 123: "NTP", 143: "IMAP", 194: "IRC", 389: "LDAP",
                443: "HTTPS", 161: "SNMP", 3306: "MySQL", 5432: "PostgreSQL", 6379: "Redis",
                1521: "Oracle DB", 3389: "RDP"
            }

            def ScanPort(port):
                try:
                    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
                        sock.settimeout(0.3)  # Timeout ajusté
                        result = sock.connect_ex((ip, port))
                        if result == 0:
                            protocol = port_protocol_map.get(port, "Unknown")
                            print(f"{BEFORE + current_time_hour() + AFTER} {ADD} Port: {white}{port}{red} Status: {white}Open{red} Protocol: {white}{protocol}{red}")
                except Exception:
                    pass

            with concurrent.futures.ThreadPoolExecutor(max_workers=max_threads) as executor:
                executor.map(ScanPort, range(1, 1025))  # scan les 1024 premiers ports par défaut

        Slow(scan_banner)
        ip = input(f"\n{BEFORE + current_time_hour() + AFTER} {INPUT} Ip -> {reset}")
        print(f"{BEFORE + current_time_hour() + AFTER} {WAIT} Scanning common ports..")
        PortScanner(ip)
        Continue()
        Reset()

    except Exception as e:
        Error(e)

def phone_number_lookup():
    Title("Phone Number Lookup")
    try:
        phone_number = input(f"\n{BEFORE + current_time_hour() + AFTER} {INPUT} Phone Number -> {color.RESET}")
        print(f"{BEFORE + current_time_hour() + AFTER} {WAIT} Information Recovery..{reset}")
        try:
            parsed_number = phonenumbers.parse(phone_number, None)
            if phonenumbers.is_valid_number(parsed_number):
                status = "Valid"
            else:
                status = "Invalid"

            country_code = "+" + phone_number[1:3] if phone_number.startswith("+") else "None"
            try: operator = carrier.name_for_number(parsed_number, "fr")
            except: operator = "None"
        
            try: type_number = "Mobile" if phonenumbers.number_type(parsed_number) == phonenumbers.PhoneNumberType.MOBILE else "Fixe"
            except: type_number = "None"

            try: 
                timezones = timezone.time_zones_for_number(parsed_number)
                timezone_info = timezones[0] if timezones else None
            except: timezone_info = "None"
                
            try: country = phonenumbers.region_code_for_number(parsed_number)
            except: country = "None"
                
            try: region = geocoder.description_for_number(parsed_number, "fr")
            except: region = "None"
                
            try: formatted_number = phonenumbers.format_number(parsed_number, phonenumbers.PhoneNumberFormat.NATIONAL)
            except: formatted_number = "None"
                
            Slow(f"""
{white}────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 {INFO_ADD} Phone        : {white}{phone_number}{red}
 {INFO_ADD} Formatted    : {white}{formatted_number}{red}
 {INFO_ADD} Status       : {white}{status}{red}
 {INFO_ADD} Country Code : {white}{country_code}{red}
 {INFO_ADD} Country      : {white}{country}{red}
 {INFO_ADD} Region       : {white}{region}{red}
 {INFO_ADD} Timezone     : {white}{timezone_info}{red}
 {INFO_ADD} Operator     : {white}{operator}{red}
 {INFO_ADD} Type Number  : {white}{type_number}{red}
{white}────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
""")
            Continue()
            Reset()
        except:
            print(f"{BEFORE + current_time_hour() + AFTER} {INFO} Invalid Format !")
            Continue()
            Reset()
    except Exception as e:
        Error(e)

def email_tracker(email):
    session = requests.Session()
    results = {}
    headers_common = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36"
    }

    # Snapchat
    try:
        headers = headers_common.copy()
        headers["Content-Type"] = "application/json"
        response = session.post(
            "https://accounts.snapchat.com/accounts/merlin/login",
            json={"email": email},
            headers=headers
        )
        # print("Snapchat response:", response.text)  # Debug si besoin
        results["Snapchat"] = "True" if "user_id" in response.text or response.status_code == 200 else "False"
    except:
        results["Snapchat"] = "Error"

    # Twitter
    try:
        headers = headers_common.copy()
        response = session.post(
            "https://api.twitter.com/i/users/email_available.json",
            data={"email": email},
            headers=headers
        )
        # print("Twitter response:", response.text)  # Debug
        json_resp = response.json()
        results["Twitter"] = "True" if json_resp.get("taken") else "False"
    except:
        results["Twitter"] = "Error"

    # Instagram
    try:
        headers = headers_common.copy()
        headers["X-Requested-With"] = "XMLHttpRequest"
        response = session.post(
            "https://www.instagram.com/accounts/account_recovery_send_ajax/",
            data={"email_or_username": email},
            headers=headers
        )
        # print("Instagram response:", response.text)  # Debug
        results["Instagram"] = "True" if response.status_code == 200 and "email" in response.text.lower() else "False"
    except:
        results["Instagram"] = "Error"

    # Facebook (méthode basique corrigée)
    try:
        headers = headers_common.copy()
        # Facebook ne fournit pas une API simple pour vérifier un email, on peut tester la page de récupération
        response = session.get("https://www.facebook.com/login/identify", headers=headers, params={"email": email})
        # print("Facebook response:", response.text)  # Debug
        results["Facebook"] = "True" if "Nous avons trouvé" in response.text or "identify your account" in response.text else "False"
    except:
        results["Facebook"] = "Error"

    # Gmail (simple test suffixe)
    try:
        results["Gmail"] = "True" if email.endswith("@gmail.com") else "False"
    except:
        results["Gmail"] = "Error"

    # Pinterest
    try:
        headers = headers_common.copy()
        response = session.get(
            "https://www.pinterest.com/_ngjs/resource/EmailExistsResource/get/",
            params={"source_url": "/", "data": '{"options": {"email": "' + email + '"}, "context": {}}'},
            headers=headers
        )
        if response.status_code == 200:
            data = response.json().get("resource_response", {})
            if data.get("message") == "Invalid email.":
                results["Pinterest"] = "False"
            else:
                results["Pinterest"] = "True" if data.get("data") is not False else "False"
        else:
            results["Pinterest"] = "Error"
    except:
        results["Pinterest"] = "Error"

    # Spotify
    try:
        headers = headers_common.copy()
        params = {'validate': '1', 'email': email}
        response = session.get('https://spclient.wg.spotify.com/signup/public/v1/account', params=params, headers=headers)
        if response.status_code == 200:
            status = response.json().get("status")
            results["Spotify"] = "True" if status == 20 else "False"
        else:
            results["Spotify"] = "Error"
    except:
        results["Spotify"] = "Error"

    # Imgur
    try:
        headers = headers_common.copy()
        headers['X-Requested-With'] = 'XMLHttpRequest'
        response = session.post('https://imgur.com/signin/ajax_email_available', headers=headers, data={'email': email})
        if response.status_code == 200:
            data = response.json().get('data', {})
            results["Imgur"] = "False" if data.get("available", False) else "True"
        else:
            results["Imgur"] = "Error"
    except:
        results["Imgur"] = "Error"

    # Reddit
    try:
        headers = headers_common.copy()
        headers["User-Agent"] = "Mozilla/5.0"
        response = session.post(
            "https://www.reddit.com/api/check_username.json",
            data={"user": email.split('@')[0]},
            headers=headers
        )
        # print("Reddit response:", response.text)  # Debug
        results["Reddit"] = "True" if response.status_code == 200 and response.json().get('available') == False else "False"
    except:
        results["Reddit"] = "Error"

    # LinkedIn
    try:
        headers = headers_common.copy()
        headers["User-Agent"] = "Mozilla/5.0"
        response = session.post(
            "https://www.linkedin.com/checkpoint/rp/request-password-reset-submit",
            data={"userName": email},
            headers=headers
        )
        # print("LinkedIn response:", response.text)  # Debug
        results["LinkedIn"] = "True" if "email" in response.text else "False"
    except:
        results["LinkedIn"] = "Error"

    # LastPass
    try:
        headers = headers_common.copy()
        params = {'check': 'avail', 'username': email}
        response = session.get('https://lastpass.com/create_account.php', params=params, headers=headers)
        if response.status_code == 200:
            results["LastPass"] = "True" if "no" in response.text else "False"
        else:
            results["LastPass"] = "Error"
    except:
        results["LastPass"] = "Error"

    # Twitch
    try:
        headers = headers_common.copy()
        headers["Client-ID"] = "kimne78kx3ncx6brgo4mv6wki5h1ko"
        response = session.get(f"https://api.twitch.tv/kraken/users?login={email.split('@')[0]}", headers=headers)
        if response.status_code == 200 and response.json().get("users"):
            results["Twitch"] = "True"
        else:
            results["Twitch"] = "False"
    except:
        results["Twitch"] = "Error"

    # Etsy (pas de vérif publique)
    results["Etsy"] = "Unknown"

    # GitHub
    try:
        headers = headers_common.copy()
        username = email.split('@')[0]
        response = session.get(f"https://github.com/{username}", headers=headers)
        results["GitHub"] = "True" if response.status_code == 200 else "False"
    except:
        results["GitHub"] = "Error"

    # Dropbox (pas de vérif publique)
    results["Dropbox"] = "Unknown"

    # Medium
    try:
        headers = headers_common.copy()
        username = email.split('@')[0]
        response = session.get(f"https://medium.com/@{username}", headers=headers)
        results["Medium"] = "True" if response.status_code == 200 else "False"
    except:
        results["Medium"] = "Error"

    # StackOverflow
    try:
        headers = headers_common.copy()
        username = email.split('@')[0]
        response = session.get(f"https://stackoverflow.com/users/{username}", headers=headers)
        results["StackOverflow"] = "True" if response.status_code == 200 else "False"
    except:
        results["StackOverflow"] = "Error"

    return results

def email_tracker_ui():
    try:
        email = input(f"\n{BEFORE + current_time_hour() + AFTER} {INPUT} Email -> {reset}")
        print(f"{BEFORE + current_time_hour() + AFTER} {WAIT} Checking Platforms..{reset}")
        results = email_tracker(email)

        for site, status in results.items():
            if status == "True":
                print(f"{BEFORE_GREEN + current_time_hour() + AFTER_GREEN} {GEN_VALID} {site}: {white}Found")
            elif status == "False":
                print(f"{BEFORE + current_time_hour() + AFTER} {GEN_INVALID} {site}: {white}Not Found")
            else:
                print(f"{BEFORE + current_time_hour() + AFTER} {ERROR} {site}: {white}{status}")

        Continue()
        Reset()
    except Exception as e:
        Error(e)

def PrintHeader():
    print(f"""
{cyan}
 ███▄ ▄███▓ ██▀███     ▓█████▄  ▒█████  ▒██   ██▒▒██   ██▒ ██▓ ███▄    █   ▄████ 
▓██▒▀█▀ ██▒▓██ ▒ ██▒   ▒██▀ ██▌▒██▒  ██▒▒▒ █ █ ▒░▒▒ █ █ ▒░▓██▒ ██ ▀█   █  ██▒ ▀█▒
▓██    ▓██░▓██ ░▄█ ▒   ░██   █▌▒██░  ██▒░░  █   ░░░  █   ░▒██▒▓██  ▀█ ██▒▒██░▄▄▄░
▒██    ▒██ ▒██▀▀█▄     ░▓█▄   ▌▒██   ██░ ░ █ █ ▒  ░ █ █ ▒ ░██░▓██▒  ▐▌██▒░▓█  ██▓
▒██▒   ░██▒░██▓ ▒██▒   ░▒████▓ ░ ████▓▒░▒██▒ ▒██▒▒██▒ ▒██▒░██░▒██░   ▓██░░▒▓███▀▒
░ ▒░   ░  ░░ ▒▓ ░▒▓░    ▒▒▓  ▒ ░ ▒░▒░▒░ ▒▒ ░ ░▓ ░▒▒ ░ ░▓ ░░▓  ░ ▒░   ▒ ▒  ░▒   ▒ 
░  ░      ░  ░▒ ░ ▒░    ░ ▒  ▒   ░ ▒ ▒░ ░░   ░▒ ░░░   ░▒ ░ ▒ ░░ ░░   ░ ▒░  ░   ░ 
░      ░     ░░   ░     ░ ░  ░ ░ ░ ░ ▒   ░    ░   ░    ░   ▒ ░   ░   ░ ░ ░ ░   ░ 
       ░      ░           ░        ░ ░   ░    ░   ░    ░   ░           ░       ░ 
                        ░                                                           
{reset}
""")

# --- Menu Principal ---
def main_menu():
    while True:
        clear_console()
        PrintHeader()              # <-- Ajout ici pour afficher le nom du tool en grand
        Title("Uruma Device - Main Menu")
        print(f"""
{cyan}
1. Instagram Lookup
2. Email Lookup
3. Dox Create
4. IP Generator
5. IP Lookup
6. IP Port Scanner
7. Phone Number Lookup
8. Email Tracker
9. Quit
{reset}
        """)
        choice = input(f"{BEFORE + current_time_hour() + AFTER} {INPUT} Choose an option (1-7) -> {reset}")

        if choice == '1':
            instagram_lookup()
        elif choice == '2':
            email_lookup()
        elif choice == '3':
            dox_create()
        elif choice == '4':
            ip_generator()
        elif choice == '5':
            ip_lookup()
        elif choice == '6':
            ip_port_scanner()
        elif choice == '7':
            phone_number_lookup()
        elif choice == '8':
            email_tracker_ui()
        elif choice == '9':
            print(f"{BEFORE + current_time_hour() + AFTER} {INFO} Goodbye!")
            break
        else:
            print(f"{BEFORE + current_time_hour() + AFTER} {ERROR} Invalid choice, please select 1-8.")
            pause()

if __name__ == "__main__":
    try:
        main_menu()
    except Exception as e:
        Error(e)
