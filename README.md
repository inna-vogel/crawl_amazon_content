# Crawl Data from Amazon

### Import libraries


```python
import csv
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import requests
import time
```

### Path to ChromeDriver


```python
options = Options()
options.add_argument("start-maximized")
driver = webdriver.Chrome(chrome_options=options, executable_path=r'C:\Driver\chromedriver.exe')
```

    

### List with the products you want to crawl


```python
# Here I was crawling th content of Graphics Cards

product_list = ['rx 580 8g', 'rx 480 8g', 'rx 590 8g', 'rx 570 8g', 'rx 470 8g', 
            'rx 490 8g', 'rx 5700 xt', 'rx 6700 xt', 'rx 5700', 'rx 6700', 
            'rx 6700 xt', 'rx 6800', 'rx 6800 xt', 'rx 6900 xt', 'vega vii', 
            'amd radeon vii', 'vega 64', 'vega 56', 'rtx 2080 ti', 'rtx 3060 ti', 
            'rtx 3070', 'rtx 3080', 'rtx 3090']
```

### Access Data from webpage


```python
# With this method you cann access the html content of a webpage

def access_data(url):    
    driver.get(url)
    soup = BeautifulSoup(driver.page_source, 'html.parser')    
    return soup
```

### Generate a URL from a search word


```python
# Generate a URL from every search word (or product word)

def generate_url(search_word):
    url = 'https://www.amazon.de/s?k={}&__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&ref=nb_sb_noss_1'
    search_word = search_word.replace(' ', '+')
    return url.format(search_word)
```

### Get next page to crawl


```python
# this method returns the next page URL

def next_page(soup):
    page = soup.find('ul', {'class': 'a-pagination'})
    if not page.find('li', {'class': 'a-disabled a-last'}):
        next_url = 'https://www.amazon.co.uk' + str(page.find('li', {'class': 'a-last'}).find('a')['href'])
        return next_url
    else:
        return
```

### Extract the content of the products


```python
# Methods extracts content of every product in the list

def extract_content(product):
    product_headline = product.h2.a
    description = product_headline.text.strip()
    product_url = 'https://www.amazon.de' + product_headline.get('href') 
    
    try:
        price_outer = product.find('span', 'a-price')
        price = price_outer.find('span', 'a-offscreen').text
        result = description, price, product_url
        print(result)
        
        #tele_url = 'https://api.telegram.org/URL_To_Telegram_Group="{}"'.format(content) #how to send information to a Telegram group you can find on my GitHub page
        #requests.get(tele_url)
        
    except AttributeError:
        return        
    
        return content
```

### Start crawling and bring it all together


```python
options = Options()
options.add_argument("start-maximized")
driver = webdriver.Chrome(chrome_options=options, executable_path=r'C:\Driver\chromedriver.exe')

records = []

for product in product_list:
    while True:
        direct_url = generate_url(product) # Generate an Amazon URL from a search word
        soup = access_data(direct_url) # Access Website
        product_content = soup.find_all('div', {'data-component-type': 's-search-result'}) #access every product in the list on the website

        for product in product_content:
            content = extract_content(product)   

            if content:
                records.append(content)
                
        product = next_page(soup) # after getting information from all products go to next site
        if not direct_url:
            break
        
#print(records)

driver.close()
```
    

    ('XFX AMD Radeon RX580 GTS XXX Edition Grafikkarte 8GB Speicher Mining, Schwarz', '620,00\xa0€', 'https://www.amazon.de/XFX-Radeon-Grafikkarte-Speicher-Schwarz/dp/B06Y66K3XD/ref=sr_1_1?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643702&sr=8-1')
    ('Memory PC Ryzen 5 3350G 4X 3.6 GHz, AMD Radeon Grafik, 8 GB DDR4, 128 GB SSD + 1000 GB, Windows 10 Pro 64bit', '449,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg1_1?ie=UTF8&adId=A00827739I6EF5NCTVVF&url=%2FMemory-PC-A10-9700-Radeon-Windows%2Fdp%2FB07G9BWW84%2Fref%3Dsr_1_9_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643702%26sr%3D8-9-spons%26psc%3D1&qualifier=1617643702&id=3552903463890881&widgetName=sp_mtf')
    ('ASRock Phantom Gaming D Radeon RX580 8G OC Grafikkarte, VR-Ready, schwarz', '792,06\xa0€', 'https://www.amazon.de/ASRock-VGA-580-Phantom-Gaming/dp/B07HKB634Y/ref=sr_1_10?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643702&sr=8-10')
    ('Yeston RX550-4G D5 LP Grafikkarte 4GB Speicher Radeon Chill GDDR5 128Bit 6000MHz VGA/DVI-D GPU', '122,99\xa0€', 'https://www.amazon.de/Yeston-RX550-4G-Grafikkarte-Speicher-6000MHz/dp/B07W4N6WYR/ref=sr_1_11?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643702&sr=8-11')
    ('Memory PC AMD Ryzen 5 4650G Pro 6X 4.2 GHz Turbo SixCore, 8 GB DDR4, 480 GB SSD, AMD Radeon Graphics, Windows 10 Pro 64bit', '529,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg1_1?ie=UTF8&adId=A01858123HPAF1WL0LP73&url=%2FMemory-PC-SixCore-Graphics-Windows%2Fdp%2FB08FJD9V1P%2Fref%3Dsr_1_14_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643702%26sr%3D8-14-spons%26psc%3D1&qualifier=1617643702&id=3552903463890881&widgetName=sp_mtf')
    ('ASRock Phantom Gaming Radeon RX550 2G Grafikkarte, schwarz', '149,00\xa0€', 'https://www.amazon.de/ASRock-90-GA0500-00UANF-VGA-Grafikkarte-Schwarz/dp/B07FLVVG6W/ref=sr_1_16?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643702&sr=8-16')
    ('RXFSP Stapelbarer offener Mining-Computer-Aluminiumrahmen für 6/8 Grafikprozessoren, Ethereum Veddha', '195,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg1_1?ie=UTF8&adId=A04413933MJOWEEA0A7D9&url=%2FRXFSP-Stapelbarer-Mining-Computer-Aluminiumrahmen-Grafikprozessoren-Ethereum%2Fdp%2FB08SC2V47C%2Fref%3Dsr_1_19_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643702%26sr%3D8-19-spons%26psc%3D1&qualifier=1617643702&id=3552903463890881&widgetName=sp_btf')
    ('Memory Gaming PC AMD Ryzen 7 2700 8X 4.1 GHz, NVIDIA GTX 1650 4GB, 16 GB DDR4, 512 GB SSD, Windows 10 Pro 64bit', '749,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg1_1?ie=UTF8&adId=A04439923ATS5KKL7Q35I&url=%2FMemory-Gaming-PC-NVIDIA-Windows%2Fdp%2FB07CL9FZ9D%2Fref%3Dsr_1_20_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643702%26sr%3D8-20-spons%26psc%3D1&qualifier=1617643702&id=3552903463890881&widgetName=sp_btf')
    ('upHere Regenbogen Bunte LED Grafikkarte GPU Brace Support-Videokarte Sehnenhalter/Holster-Halterung,Unterstützung für Länge und Höhe einstellbar,Regenbogen Bunte LED,GS05CF', '14,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg1_1?ie=UTF8&adId=A01766282156YFXZAGW9W&url=%2FupHere-Support-Videokarte-Sehnenhalter-Holster-Halterung-Unterst%25C3%25BCtzung%2Fdp%2FB082SQK3X2%2Fref%3Dsr_1_21_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643702%26sr%3D8-21-spons%26psc%3D1&qualifier=1617643702&id=3552903463890881&widgetName=sp_btf')
    ('Memory PC Ryzen 5 3350G 4X 3.6 GHz, AMD Radeon Grafik, 8 GB DDR4, 240 GB SSD ohne Windows', '419,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg2_1?ie=UTF8&adId=A0408229NIEK1POFP8YQ&url=%2FMemory-PC-A10-9700-Radeon-Windows%2Fdp%2FB07G98BZMK%2Fref%3Dsr_1_17_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643704%26sr%3D8-17-spons%26psc%3D1&qualifier=1617643704&id=8498224393424633&widgetName=sp_atf_next')
    ('Komplett PC-Paket Ryzen 3 4350G Pro 4X 4.0 GHz Turbo, 16 GB DDR4, 240 GB SSD + 2000 GB, Radeon Grafik, Windows 10 Pro 64bit + 22 Zoll TFT + Tastatur Set + W-LAN Bundle', '689,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg2_1?ie=UTF8&adId=A0187876HCTX591VAPIS&url=%2FKomplett-PC-Paket-Radeon-Windows-Tastatur%2Fdp%2FB08J7RZ9B6%2Fref%3Dsr_1_18_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643704%26sr%3D8-18-spons%26psc%3D1&qualifier=1617643704&id=8498224393424633&widgetName=sp_atf_next')
    ('ASUS NVIDIA GeForce GT 710 Silent Grafikkarte (2GB DDR5 Speicher, 0dB Kühlung, DVI, VGA, HDMI)', '57,99\xa0€', 'https://www.amazon.de/ASUS-90YV0AL1-M0NA00-2GB-Grafikkarte-schwarz/dp/B07489XSJP/ref=sr_1_19?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643704&sr=8-19')
    ('Asus PH-GTX1050TI-4G Nvidia GeForce Grafikkarte (PCIe 3.0, 4GB DDR5 Speicher, HDMI, DVI, DisplayPort)', '307,00\xa0€', 'https://www.amazon.de/PH-GTX1050TI-4G-GeForce-Grafikkarte-Speicher-DisplayPort/dp/B01MFBKRI5/ref=sr_1_20?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643704&sr=8-20')
    ('MSI GeForce GT 1030 2GH LP OC 2GB Nvidia GDDR5 1x HDMI, 1x DP, 2 Slot Low Proflie, Afterburner OC, Heat sink Grafikkarte', '106,43\xa0€', 'https://www.amazon.de/MSI-GeForce-Proflie-Afterburner-Grafikkarte/dp/B071NW37J8/ref=sr_1_24?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643704&sr=8-24')
    ('Asus Phoenix GeForce PH-GT1030-O2G Grafikkarte (Nvidia, PCIe 3.0, 2GB GDDR5 Speicher, HDMI, DVI)', '119,90\xa0€', 'https://www.amazon.de/Phoenix-GeForce-PH-GT1030-O2G-Grafikkarte-Speicher/dp/B0727WGG3F/ref=sr_1_26?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643704&sr=8-26')
    ('Corsair ONE a100 Kompakter Gaming-PC – AMD Ryzen 9 3950X-CPU – NVIDIA GeForce RTX 2080 Ti-Grafikkarte – 32 GB CORSAIR VENGEANCE LPX DDR4-Arbeitsspeicher', '4.301,98\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg2_1?ie=UTF8&adId=A04653313QNP6B3KBA41B&url=%2FCorsair-a100-Kompakter-Gaming-PC-DDR4-Arbeitsspeicher%2Fdp%2FB08CDQY9PC%2Fref%3Dsr_1_27_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643704%26sr%3D8-27-spons%26psc%3D1&qualifier=1617643704&id=8498224393424633&widgetName=sp_mtf')
    ('lzndeal 6 GPU Mining Rig Aluminiumgehäuse + 4 Lüfter Open Air Frame Stapelbarer Mining Case Rig Open Air Frame für ETH ZEC/Bitcoin Crypto Coin Currency Mining', '79,69\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg2_1?ie=UTF8&adId=A01511293BPTEWWP8AWYP&url=%2Flzndeal-Aluminiumgeh%25C3%25A4use-Stapelbarer-Bitcoin-Currency%2Fdp%2FB08VGFXPB8%2Fref%3Dsr_1_28_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643704%26sr%3D8-28-spons%26psc%3D1&qualifier=1617643704&id=8498224393424633&widgetName=sp_mtf')
    ('ASUS Nvidia GT1030 2GB BRK Low Profile Gaming Grafikkarte (GDDR5 Speicher, PCIe 3.0, ,DVI, HDMI, Passiv, GT1030-2G-BRK)', '126,40\xa0€', 'https://www.amazon.de/ASUS-90YV0AT2-M0NA00-Grafikkarte-2GB-schwarz/dp/B0789CF1KM/ref=sr_1_29?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643704&sr=8-29')
    ('AMD Radeon Pro WX 3100\xa04\xa0GB GDDR5\xa0–\xa0Graphics Cards (Radeon Pro WX 3100, 4\xa0GB, GDDR5, 128\xa0Bit, 1500\xa0MHz, PCI Express x16)', '197,58\xa0€', 'https://www.amazon.de/AMD-Graphics-128-Bit-1500-MHz-Express/dp/B073Q1SBB9/ref=sr_1_30?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643704&sr=8-30')
    ('Asus STRIX-GTX1050TI-O4G-GAMING Grafikkarte (Nvidia, PCIe 3.0, 4GB DDR5 Speicher, HDMI, DVI, Displayport)', '470,96\xa0€', 'https://www.amazon.de/STRIX-GTX1050TI-O4G-GAMING-Grafikkarte-Nvidia-Speicher-Displayport/dp/B01M3TR08L/ref=sr_1_34?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643704&sr=8-34')
    ('KFA2 nVidia GeForce GTX 1660 Super OC 6GB 192-bit GDDR6 PCIe Grafikkarte, 60SRL7DSY91K, schwarz', '722,22\xa0€', 'https://www.amazon.de/KFA2-GTX-1660-Super-60SRL7DSY91K/dp/B07ZJMRF2K/ref=sr_1_35?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643704&sr=8-35')
    ('Ledger Nano S – Die Beste Crypto-Hardware-Geldbörse – Sichern und verwalten Sie Ihre Bitcoin, Ethereum, ERC20 und viele andere Münzen', '59,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg2_1?ie=UTF8&adId=A01319033VQ2J9RCEUWKV&url=%2FLedger-Nano-Cryptocurrency-Hardware-Geldb%25C3%25B6rse-Ethereum-Altcoins%2Fdp%2FB07FY5R77T%2Fref%3Dsr_1_37_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643704%26sr%3D8-37-spons%26psc%3D1&qualifier=1617643704&id=8498224393424633&widgetName=sp_btf')
    ('lzn 6 GPU Stahlmünze Open Air Miner Bergbau Rahmen Rig Fall bis zu 6 GPU BTC LTC ETH Ethereum Neu', '49,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg2_1?ie=UTF8&adId=A00842992S0HE2QGCQPWS&url=%2Flzn-Stahlm%25C3%25BCnze-Bergbau-Rahmen-Ethereum%2Fdp%2FB078GL8SG5%2Fref%3Dsr_1_38_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643704%26sr%3D8-38-spons%26psc%3D1&qualifier=1617643704&id=8498224393424633&widgetName=sp_btf')
    ('RXFSP Stapelbarer offener Mining-Computer-Aluminiumrahmen für 6/8 Grafikprozessoren, Ethereum Veddha', '165,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg3_1?ie=UTF8&adId=A0037163JBMZMT3MA11H&url=%2FRXFSP-Stapelbarer-Mining-Computer-Aluminiumrahmen-Grafikprozessoren-Ethereum%2Fdp%2FB08TMPVQVS%2Fref%3Dsr_1_33_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643706%26sr%3D8-33-spons%26psc%3D1&qualifier=1617643706&id=4052817647550179&widgetName=sp_atf_next')
    ('DMP Monitor Tischhalterung LCD 352S - Halterung für Monitore bis zu 32 Zoll (81, 3cm max. 10kg) - neigbar schwenkbar höhenverstellbar VESA 75/100 schwarz', '29,90\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg3_1?ie=UTF8&adId=A0863897L94YKHU3FGED&url=%2FMonitor-Tischhalterung-LCD-352S-h%25C3%25B6henverstellbar-schwarz%2Fdp%2FB00BHFHLZO%2Fref%3Dsr_1_34_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643706%26sr%3D8-34-spons%26psc%3D1&qualifier=1617643706&id=4052817647550179&widgetName=sp_atf_next')
    ('Corsair AX1600i digitales PC-Netzteil (Voll-Modulares Kabelmanagement, 80 Plus Titanium, 1600 Watt, EU)', '670,62\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg3_1?ie=UTF8&adId=A01363801KDJX68353ET5&url=%2FCorsair-digitales-PC-Netzteil-Voll-Modulares-Kabelmanagement%2Fdp%2FB078W6V4RX%2Fref%3Dsr_1_43_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643706%26sr%3D8-43-spons%26psc%3D1&qualifier=1617643706&id=4052817647550179&widgetName=sp_mtf')
    ('Jaimenalin 6 StüCke Neueste Ver009 USB 3.0 Pci-E Riser 009 S Express 1X4X8X16X Extender Riser Karte Sata 15Pin zu 6 Pin Stromkabel', '31,81\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg3_1?ie=UTF8&adId=A0644914OLCEE6469546&url=%2FJaimenalin-Neueste-1X4X8X16X-Extender-Stromkabel-3-farben%2Fdp%2FB08NJL44NF%2Fref%3Dsr_1_44_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643706%26sr%3D8-44-spons%26psc%3D1&qualifier=1617643706&id=4052817647550179&widgetName=sp_mtf')
    ('Gigabyte AORUS Radeon RX 6900 XT Master Grafikkarte, 16 GB', '2.625,04\xa0€', 'https://www.amazon.de/Gigabyte-AORUS-Radeon-Master-Grafikkarte/dp/B08WBYGJ83/ref=sr_1_48?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643706&sr=8-48')
    ('Sun3Drucker EXP GDC Laptop Externe unabhängige Grafikkarte PCI-E-Grafikkarte für Beast Dock Mini PCI-E', '44,98\xa0€', 'https://www.amazon.de/Sun3Drucker-Laptop-External-Independent-Graphics/dp/B073YQT41T/ref=sr_1_51?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643706&sr=8-51')
    ('YWZQ Mining Power 1800W Modulares Mining-Netzteil für 8 GPUs ETH Rig Ethereum Miner', '142,33\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg3_1?ie=UTF8&adId=A015527410VMI1CS5AFB8&url=%2FYWZQ-Mining-Modulares-Mining-Netzteil-Ethereum%2Fdp%2FB08QYQD5HN%2Fref%3Dsr_1_53_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643706%26sr%3D8-53-spons%26psc%3D1&qualifier=1617643706&id=4052817647550179&widgetName=sp_btf')
    ('KFA2 nVidia GeForce GTX 1660 Super OC 6GB 192-bit GDDR6 PCIe Grafikkarte, 60SRL7DSY91K, schwarz', '722,22\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg3_1?ie=UTF8&adId=A046906831J4CIJGQUTWW&url=%2FKFA2-GTX-1660-Super-60SRL7DSY91K%2Fdp%2FB07ZJMRF2K%2Fref%3Dsr_1_54_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643706%26sr%3D8-54-spons%26psc%3D1&qualifier=1617643706&id=4052817647550179&widgetName=sp_btf')
    ('HUANUO 13-27 Zoll Monitor Halterung 2 Monitore, Volleinstellbar für LCD LED Bildschirme, 2 Montageoptionen, VESA 75/100, Schwarz', '36,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg4_1?ie=UTF8&adId=A0175912WQ7LK9ZC1PWM&url=%2FHUANUO-Halterung-Volleinstellbar-Bildschirme-Montageoptionen-Schwarz%2Fdp%2FB07PWK4R9S%2Fref%3Dsr_1_49_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643708%26sr%3D8-49-spons%26psc%3D1%26smid%3DA2OPPOG3I7NMUH&qualifier=1617643708&id=8782083239158966&widgetName=sp_atf_next')
    ('1m Adressierbar RGB LED Streifen, 5V, 32 LED/m, Wasserfest, WS2801 Ganzes 24-Bit Farbe, 4-polig JST-SM Steckverbinder vorgelötet Zu Both Stützen', '24,95\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg4_1?ie=UTF8&adId=A04396382D78AXIBJRQMZ&url=%2FAdressierbar-Streifen-Wasserfest-Steckverbinder-vorgel%25C3%25B6tet%2Fdp%2FB008F05N54%2Fref%3Dsr_1_50_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643708%26sr%3D8-50-spons%26psc%3D1%26smid%3DA1TIGMJ4AZ1K1A&qualifier=1617643708&id=8782083239158966&widgetName=sp_atf_next')
    ('ASUS Radeon RX 580 O4G Dual-Fan OC Edition GDDR5 DP HDMI DVI VR Ready AMD Grafikkarte (DUAL-RX580-O4G)', '922,31\xa0€', 'https://www.amazon.de/ASUS-580-T8G-GDDR5-DP-Grafikkarte-rog-strix-rx580-t8g-gaming/dp/B071CMPRZZ/ref=sr_1_52?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643708&sr=8-52')
    ('Corsair RM850 PC-Netzteil (Voll-Modulares Kabelmanagement, 80 Plus Gold, 850 Watt, EU) schwarz', '124,90\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg4_1?ie=UTF8&adId=A05050982X5ZBJU1N3WBN&url=%2FCorsair-PC-Netzteil-Voll-Modulares-Kabelmanagement-schwarz%2Fdp%2FB07S3X7QV7%2Fref%3Dsr_1_59_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643708%26sr%3D8-59-spons%26psc%3D1&qualifier=1617643708&id=8782083239158966&widgetName=sp_mtf')
    ('KFA2 26NRL7HPX7OK nVidia GeForce RTX 2060 OC 6GB 256-bit GDDR6 PCIe Grafikkarte, schwarz', '822,22\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg4_1?ie=UTF8&adId=A04689441WVJFP301XLWO&url=%2FKFA2-GeForce-2060-192-bit-GDDR6%2Fdp%2FB07M85G6PD%2Fref%3Dsr_1_60_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643708%26sr%3D8-60-spons%26psc%3D1&qualifier=1617643708&id=8782083239158966&widgetName=sp_mtf')
    ('Razer Core X Externes Grafikkarten Gehäuse mit Thunderbolt 3 für Windows 10 and Mac Laptops, Schwarz', '279,91\xa0€', 'https://www.amazon.de/Razer-Externes-Grafikkarten-Geh%C3%A4use-Thunderbolt/dp/B07D4NBPBC/ref=sr_1_61?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643708&sr=8-61')
    ('Docooler Laptop Externe Unabhängige Grafikkarte Grafik Dock Expresscard Version/Key Version/Mini PCI-E Version für V8.0 EXP GDC Beast', '41,99\xa0€', 'https://www.amazon.de/Docooler-Externe-Unabh%C3%A4ngige-Grafikkarte-Expresscard-Version/dp/B07BZFV827/ref=sr_1_62?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643708&sr=8-62')
    ('Festnight Laptop Externe unabhängige Grafikkarte Grafik Dock NGFF M.2 A/E Schlüssel Version für V8.0 EXP GDC Beast', '48,14\xa0€', 'https://www.amazon.de/Festnight-Externe-unabh%C3%A4ngige-Grafikkarte-Schl%C3%BCssel-Grey/dp/B07SLXDVF5/ref=sr_1_64?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643708&sr=8-64')
    ('Dell Radeon Pro WX 2100, 2GB, DP. 2 mDP, (Precision) (Kunden-Set) *Wie 490-BDZR *', '145,46\xa0€', 'https://www.amazon.de/Dell-Radeon-WX2100-2mDP-490-BDZR/dp/B079Y88M1L/ref=sr_1_65?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643708&sr=8-65')
    ('AMD FIREPRO 2270\xa0Grafikkarte ATI 400\xa0MHz pci-ex16\xa01024\xa0MB', '35,52\xa0€', 'https://www.amazon.de/AMD-FIREPRO-2270-Grafikkarte-400-MHz-pci-ex16-1024-MB/dp/B000EWEPB4/ref=sr_1_66?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643708&sr=8-66')
    ('AGANDO Aqua Gaming PC - Be Quiet! 500DX,MSI B550,32 GB DDR,ASUS TUF RTX3080 OC,500 GB M.2 SSD+ WD 2TB, Win 10 Pro', '2.999,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg4_1?ie=UTF8&adId=A01100263DGDOPFI88MWP&url=%2FAGANDO-Aqua-Gaming-PC-RTX3080%2Fdp%2FB08ZDVFVRW%2Fref%3Dsr_1_69_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643708%26sr%3D8-69-spons%26psc%3D1&qualifier=1617643708&id=8782083239158966&widgetName=sp_btf')
    ('Breakout Board für HP Server Netzteile GPU Mining mit EIN/Aus-Schalter 17x 6Pin PCI-e Slots', '10,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg4_1?ie=UTF8&adId=A00684583II1NKLQ7DUH3&url=%2Fperfk-Breakout-Server-Netzteile-aus-Schalter-als-Bild%2Fdp%2FB07G16T2CC%2Fref%3Dsr_1_70_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643708%26sr%3D8-70-spons%26psc%3D1&qualifier=1617643708&id=8782083239158966&widgetName=sp_btf')
    ('lzn PCI-E Riser Karte 1x zu 16x USB 3.0 VER 009 S Bergbau Extender Board für Bitcoin 1 stück', '6,59\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg5_1?ie=UTF8&adId=A08598871XI5Q68QCM2LS&url=%2Flzn-Bergbau-Extender-Bitcoin-st%25C3%25BCcke%2Fdp%2FB078TDYL6B%2Fref%3Dsr_1_65_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643713%26sr%3D8-65-spons%26psc%3D1&qualifier=1617643713&id=6708169818224822&widgetName=sp_atf_next')
    ('Montech Air X ARGB– Preisgekröntes ARGB Mid-Tower ATX-Gaming-Gehäuse mit gehärteter Glasseite, 3 eingebaute ARGB-Lüfter, kompaktes Gehäuse - Kompaktes ATX PC Gehäuse weiß', '80,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg5_1?ie=UTF8&adId=A099243013A4OAL4WAQRG&url=%2FMontech-Preisgekr%25C3%25B6ntes-ATX-Gaming-Geh%25C3%25A4use-geh%25C3%25A4rteter-ARGB-L%25C3%25BCfter%2Fdp%2FB084QN3Z26%2Fref%3Dsr_1_66_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643713%26sr%3D8-66-spons%26psc%3D1&qualifier=1617643713&id=6708169818224822&widgetName=sp_atf_next')
    ('PNY GeForce GT 710 GF710GTLH2GEPB 2048MB DDR3 PCI-e 2.0 HD Grafikkarte', '56,86\xa0€', 'https://www.amazon.de/PNY-GeForce-GF710GTLH2GEPB-2048MB-Grafikkarte/dp/B01IC0FRFW/ref=sr_1_67?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643713&sr=8-67')
    ('Festnight Laptop Externe unabhängige Grafikkarte Grafik Dock Mini PCI-E-Version für V8.0 EXP GDC Beast', '36,94\xa0€', 'https://www.amazon.de/Festnight-Externe-unabh%C3%A4ngige-Grafikkarte-Version-Grey/dp/B07SLXDVYR/ref=sr_1_69?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643713&sr=8-69')
    ('iHaospace 95MM T129215SM FDC10M12S9-C 4Pin 12V Graphics Card Cooler Fan for ASUS ROG Strix GTX 1050Ti 1060Ti 1070Ti 1080Ti RX 470 570 580 RX470 RX570 RX580 Grafikkarte Lüfter', '24,99\xa0€', 'https://www.amazon.de/iHaospace-T129215SM-FDC10M12S9-C-Graphics-1050Ti/dp/B07FKCN6TT/ref=sr_1_71?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643713&sr=8-71')
    ('CORSAIR Hydro X Series XG7 RGB RX-Series GPU-Wasserkühler (6900 XT, 6800 XT)', '202,35\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg5_1?ie=UTF8&adId=A05810173B1W9BHP4REX7&url=%2FCORSAIR-Hydro-GPU-Wasserk%25C3%25BChler-6900-6800%2Fdp%2FB08TRK49WK%2Fref%3Dsr_1_75_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643713%26sr%3D8-75-spons%26psc%3D1&qualifier=1617643713&id=6708169818224822&widgetName=sp_mtf')
    ('AGANDO Aqua Highend Business / Gaming PC , Intel i9 10900K, RTX 3090 24G , 32 GB DDR , 500 GB M.2 HDD, Win10 Pro', '3.999,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg5_1?ie=UTF8&adId=A010987021ZWZU66OTT05&url=%2FAGANDO-Highend-Business-Gaming-10900K%2Fdp%2FB08XBZT6X7%2Fref%3Dsr_1_76_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643713%26sr%3D8-76-spons%26psc%3D1&qualifier=1617643713&id=6708169818224822&widgetName=sp_mtf')
    ('SODIAL PCIe PCI-E V8.4D EXP GDC Externe Laptop Grafikkarte Dock/Laptop Dockingstation (Mini PCI-E Schnittstelle Version)', '33,96\xa0€', 'https://www.amazon.de/SODIAL-Externe-Grafikkarte-Dockingstation-Schnittstelle-schwarz/dp/B07KVZ8BLV/ref=sr_1_78?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643713&sr=8-78')
    ('Asus ROG Strix GeForce GTX1070-O8G Gaming Grafikkarte (Nvidia, PCIe 3.0, 8GB GDDR5 Speicher, HDMI, DVI, DisplayPort) (Generalüberholt)', '955,55\xa0€', 'https://www.amazon.de/GTX1070-O8G-Grafikkarte-Speicher-DisplayPort-General%C3%BCberholt/dp/B082WJR22T/ref=sr_1_79?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643713&sr=8-79')
    ('Sapphire 34263-02-52G Gearbox Thunderbolt Retail Grafikkarten', '329,54\xa0€', 'https://www.amazon.de/Sapphire-34263-02-52G-Gearbox-Thunderbolt-Grafikkarten/dp/B07NPGYNC2/ref=sr_1_80?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643713&sr=8-80')
    ('KIKIRon Grafikkarte 6GB GDDR5 192bit-1860MHz 8Gbps Gaming Video-Grafikkarte (Farbe : Schwarz, Größe : Einheitsgröße)', '605,47\xa0€', 'https://www.amazon.de/KIKIRon-Grafikkarte-192bit-1860MHz-Gaming-Video-Grafikkarte/dp/B085T7NHTN/ref=sr_1_82?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643713&sr=8-82')
    ('POS-Cardsysteme EQT-410 Cash Drawer, schwarz', '89,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg5_1?ie=UTF8&adId=A018368131WUGR4QGKKDV&url=%2FPOS-Cardsysteme-EQT-410-Cash-Drawer-schwarz%2Fdp%2FB01E5C7UTW%2Fref%3Dsr_1_85_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643713%26sr%3D8-85-spons%26psc%3D1&qualifier=1617643713&id=6708169818224822&widgetName=sp_btf')
    ('1TB, 1000GB Festplatte kompatibel für Toshiba Satellite P300 25V', '64,90\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg5_1?ie=UTF8&adId=A03352583CDZVEZRDT8U1&url=%2F1000GB-Festplatte-kompatibel-Toshiba-Satellite%2Fdp%2FB014WFLVDI%2Fref%3Dsr_1_86_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643713%26sr%3D8-86-spons%26psc%3D1&qualifier=1617643713&id=6708169818224822&widgetName=sp_btf')
    ('Silverbead Wärmeleitpaste [3 Stück] Hochleistungs Thermische Thermal Paste Grease Compound TC-009 | für alle CPU Kühler PC GPU LED SMD IC Heatsink Kühlkörper Wasserkühlung Notebook & Laptop PS4', '3,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg6_1?ie=UTF8&adId=A091797627D96BQZ5JEEY&url=%2FBONYX-W%25C3%25A4rmeleitpaste-Thermische-Thermal-Heatsink%2Fdp%2FB01ATG0P04%2Fref%3Dsr_1_81_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643715%26sr%3D8-81-spons%26psc%3D1&qualifier=1617643715&id=5785713649600103&widgetName=sp_atf_next')
    ('upHere Grafikkarte GPU Brace Support-Videokarte Sehnenhalter/Holster-Halterung,Einzelsteckkarten (Schwarz),G276BK', '11,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg6_1?ie=UTF8&adId=A08743331XK4TYKNQ65ZQ&url=%2FupHere-Support-Videokarte-Holster-Halterung-Raumfahrt-Aluminium-Einzelsteckkarten%2Fdp%2FB08HLQ93F8%2Fref%3Dsr_1_82_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643715%26sr%3D8-82-spons%26psc%3D1&qualifier=1617643715&id=5785713649600103&widgetName=sp_atf_next')
    ('KIKIRon Grafikkarte 6GB GDDR5 192bit 1785MHz 8Gbps Gaming-Grafikkarte (Farbe : Schwarz, Größe : Einheitsgröße)', '620,59\xa0€', 'https://www.amazon.de/KIKIRon-Grafikkarte-192bit-1785MHz-Gaming-Grafikkarte/dp/B085STDDR4/ref=sr_1_83?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643715&sr=8-83')
    ('HP NVIDIA Quadro RTX 4000 8GB (3) DP+USBc - Grafikkarte, 5JV89AA', '1.058,03\xa0€', 'https://www.amazon.de/HP-NVIDIA-Quadro-4000-USBc/dp/B07PXP8H3N/ref=sr_1_86?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643715&sr=8-86')
    ('HP 5JH81AA Grafikkarten,', '3.504,29\xa0€', 'https://www.amazon.de/HP-NVIDIA-Quadro-5000-16GB/dp/B07PJSNX3F/ref=sr_1_87?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643715&sr=8-87')
    ('Montech Air X ARGB preisgekröntes Mid-Tower-Gaming-Gehäuse mit gehärtetem Glas an der Seite, Perfekter Staubschutz 3 eingebaute ARGB-Lüfter, kompaktes Gehäuse, tolles ATX-PC-Gehäuse Schwarz', '79,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg6_1?ie=UTF8&adId=A0993566BE3JN7IQT4LD&url=%2FMontech-ARGB-Kompaktes-Geh%25C3%25A4use-schwarz%2Fdp%2FB084Q8PHPW%2Fref%3Dsr_1_91_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643715%26sr%3D8-91-spons%26psc%3D1&qualifier=1617643715&id=5785713649600103&widgetName=sp_mtf')
    ('Corsair RM650x PC-Netzteil (Voll-Modulares Kabelmanagement, 80 Plus Gold, 650 Watt, EU)', '109,90\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg6_1?ie=UTF8&adId=A00081021O03HN10CTQXM&url=%2FCorsair-RM650x-PC-Netzteil-Voll-Modulares-Kabelmanagement%2Fdp%2FB07CJ3Y52K%2Fref%3Dsr_1_92_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643715%26sr%3D8-92-spons%26psc%3D1&qualifier=1617643715&id=5785713649600103&widgetName=sp_mtf')
    ('Bewinner Laptop External Independent-Grafikkartendock für Mini-PCI-E - Unterstützung für 6Pin + 8Pin-Schnittstelle für Engineering-Messungen, Datenerfassung, Server-Debugging, GPU-Computing', '53,99\xa0€', 'https://www.amazon.de/Bewinner-Laptop-External-Independent-Grafikkartendock-Mini-PCI-default/dp/B07NRY1MCH/ref=sr_1_95?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643715&sr=8-95')
    ('MSI GeForce GTX 1070 SEA HAWK X 8GB Nvidia GDDR5 1x HDMI, 3x DP, 1x DL-DVI-D, 2 Slot Afterburner OC, VR Ready, 4K-optimiert, Grafikkarte', '1.111,00\xa0€', 'https://www.amazon.de/MSI-DL-DVI-D-Afterburner-4K-optimiert-Grafikkarte/dp/B01HBG0VY0/ref=sr_1_96?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643715&sr=8-96')
    ('Razer Gehäuse für Externe Grafikkarte Schwarz', '361,92\xa0€', 'https://www.amazon.de/Razer-Geh%C3%A4use-Externe-Grafikkarte-Schwarz/dp/B07D9CD8M9/ref=sr_1_98?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643715&sr=8-98')
    ('PNY VCQ295NVS-X16 nVidia Quadro NVS 295 PCI Express x16 Dual Display Desktop Grafikkarte', '398,00\xa0€', 'https://www.amazon.de/vcq295nvs-x16-NVIDIA-295-PCI-Express-x16-Dual-Grafikkarte/dp/B07D85HKZJ/ref=sr_1_99?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643715&sr=8-99')
    ('6 GPU Stahlmünzen Open Air Miner Mining Rig Fallen Sie auf GPU für Crypto Coin Currency Mining', '50,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg6_1?ie=UTF8&adId=A03403651MU4GLH2ISCJ1&url=%2FStahlm%25C3%25BCnzen-Mining-Fallen-Crypto-Currency%2Fdp%2FB08T6MTFQ1%2Fref%3Dsr_1_101_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643715%26sr%3D8-101-spons%26psc%3D1&qualifier=1617643715&id=5785713649600103&widgetName=sp_btf')
    ('Gainward GRA PCX GT730 SilentFX Grafikkarte (PCI-e, 2GB GDDR3-Speicher, HDMI, DVI, VGA, 1 GPU)', '74,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg6_1?ie=UTF8&adId=A0691928RPMPZKLMK1PA&url=%2FGainward-GT730-SilentFX-Grafikkarte-GDDR3-Speicher%2Fdp%2FB00L2GTSKO%2Fref%3Dsr_1_102_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643715%26sr%3D8-102-spons%26psc%3D1%26smid%3DA2QBPT2NRCMPLN&qualifier=1617643715&id=5785713649600103&widgetName=sp_btf')
    ('PNY NVIDIA QUADRO K1200 professionelle Grafikkarte 4 GB GDDR5 PCI-Express Low Profile 4K 4 x DP (VCQK1200DP-PB)', '498,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg7_1?ie=UTF8&adId=A08796023U5QAHAVH6KED&url=%2FPNY-professionelle-Grafikkarte-PCI-Express-VCQK1200DP-PB%2Fdp%2FB00V9SZPOS%2Fref%3Dsr_1_97_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643717%26sr%3D8-97-spons%26psc%3D1&qualifier=1617643717&id=5083770642212450&widgetName=sp_atf_next')
    ('Lenovo Grafikkarte Quadro RTX 4000-8 GB GDDR6 PCIe 3.0 X16-3 X DisplayPort, USB-C', '1.298,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg7_1?ie=UTF8&adId=A001597910IA455182AD7&url=%2FLenovo-Quadro-4000-8-GDDR6-DisplayPort%2Fdp%2FB07TXRHYW1%2Fref%3Dsr_1_98_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643717%26sr%3D8-98-spons%26psc%3D1&qualifier=1617643717&id=5083770642212450&widgetName=sp_atf_next')
    ('StarTech.com PCI Express auf 2 PCI & 2 PCIe Erweiterungsgehäuse', '370,62\xa0€', 'https://www.amazon.de/PCI-Express-auf-PCIe-Erweiterungsgeh%C3%A4use/dp/B002I9SK5S/ref=sr_1_99?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643717&sr=8-99')
    ('Camisin 87MM PLD09210S12M PLD09210S12HH LüFter Ersetzen Sie Den KüHler für Den Strix GTX 1060 OC 1070 1080 GTX 1080Ti RX 480-BildkartenlüFter', '18,07\xa0€', 'https://www.amazon.de/Adanse-PLD09210S12M-PLD09210S12HH-L%C3%BCfter-Bildkartenventilator-Schwarz/dp/B0855PSBKS/ref=sr_1_100?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643717&sr=8-100')
    ('Beada PLD09210S12M PLD09210S12HH LüFter Ersatz Den KüHler für Strix GTX1060 OC 1070 1080 GTX1080Ti RX480 Grafik Karte LüFter', '13,07\xa0€', 'https://www.amazon.de/Beada-PLD09210S12M-PLD09210S12HH-GTX1060-GTX1080Ti-schwarz/dp/B091KSGCRJ/ref=sr_1_101?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643717&sr=8-101')
    ('Asus Dual-RX460-2G AMD Radeon Grafikkarte (2GB DDR5 Speicher, PCIe 3.0, HDMI, DisplayPort, DVI)', '278,00\xa0€', 'https://www.amazon.de/Dual-RX460-2G-Radeon-Grafikkarte-Speicher-DisplayPort/dp/B01LQ1Z9D4/ref=sr_1_103?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643717&sr=8-103')
    ('ACAMPTAR 87MM PLD09210S12M PLD09210S12HH LüFter Ersetzen Sie Den KüHler für Den Strix GTX 1060 OC 1070 1080 GTX 1080Ti RX 480-BildkartenlüFter', '14,92\xa0€', 'https://www.amazon.de/ACAMPTAR-PLD09210S12M-PLD09210S12HH-Ersetzen-480-Bildkartenl%C3%BCFter-schwarz/dp/B08B5VW1K4/ref=sr_1_104?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643717&sr=8-104')
    ('upHere Grafikkarte GPU Brace Support-Videokarte Sehnenhalter/Holster-Halterung,Einzelsteckkarten (Weiß),G276WT', '11,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg7_1?ie=UTF8&adId=A0874145N9GFVQRG76KA&url=%2FupHere-Support-Videokarte-Holster-Halterung-Raumfahrt-Aluminium-Einzelsteckkarten%2Fdp%2FB08HLSFQBP%2Fref%3Dsr_1_107_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643717%26sr%3D8-107-spons%26psc%3D1&qualifier=1617643717&id=5083770642212450&widgetName=sp_mtf')
    ('Sharkoon RGB LIT 100, PC Gehäuse', '62,90\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg7_1?ie=UTF8&adId=A023156323INIDH2WP1EO&url=%2FSharkoon-RGB-LIT-100-Geh%25C3%25A4use%2Fdp%2FB08296VX7R%2Fref%3Dsr_1_108_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643717%26sr%3D8-108-spons%26psc%3D1&qualifier=1617643717&id=5083770642212450&widgetName=sp_mtf')
    ('Corsair Carbide Series Air 540 PC-Gehäuse (ATX High Airflow, Dual-Kammer, geeignet für ATX, Micro ATX, E-ATX, and Mini ITX, mit Lüfter) schwarz', '125,38\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg7_1?ie=UTF8&adId=A10121751XUVNBLGVILQ&url=%2FCorsair-Carbide-PC-Geh%25C3%25A4use-Dual-Kammer-geeignet%2Fdp%2FB00D6GINF4%2Fref%3Dsr_1_117_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643717%26sr%3D8-117-spons%26psc%3D1&qualifier=1617643717&id=5083770642212450&widgetName=sp_btf')
    ('Nvidia Quadro K5200 8GB 256-Bit PCIe x16 Computer Grafikkarte GPU Dell R93GX', '998,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg7_1?ie=UTF8&adId=A08573582ZSM1I79CS60V&url=%2FNVIDIA-QUADRO-K5200-256-bit-x16-Computer%2Fdp%2FB01BI5X1TC%2Fref%3Dsr_1_118_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643717%26sr%3D8-118-spons%26psc%3D1&qualifier=1617643717&id=5083770642212450&widgetName=sp_btf')
    ('Sidorenko Gaming Mauspad - 280 x 200 mm - Vernähte Kanten - Rutschfest - Mousepad mit einer speziellen Oberfläche verbessert Geschwindigkeit und Präzision | schwarz', '7,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg8_1?ie=UTF8&adId=A016390210TVGVMGUBDU4&url=%2FSidorenko-Fransenfreie-Oberfl%25C3%25A4che-verbessert-Geschwindigkeit%2Fdp%2FB07G41Q2VL%2Fref%3Dsr_1_113_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643719%26sr%3D8-113-spons%26psc%3D1&qualifier=1617643719&id=8741623108871404&widgetName=sp_atf_next')
    ('Eightwood RP-SMA Kabel PCI IPX/u.FL auf RP-SMA Buchse 1.13mm 8inch 20cm für AVM PCB Wireless LAN WLAN Router Laptop WiFi Card MEHRWEG', '6,69\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_next_aps_sr_pg8_1?ie=UTF8&adId=A02656171T5422Y1M5XNP&url=%2FEightwood-RP-SMA-Antenneadapter-Pigtail-Wireless%2Fdp%2FB01G6E5XLE%2Fref%3Dsr_1_114_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643719%26sr%3D8-114-spons%26psc%3D1&qualifier=1617643719&id=8741623108871404&widgetName=sp_atf_next')
    ('KIKIRon Grafikkarte 6G-Spiel-Videografikkarte 128bit 1530MHz 12000MHz GDDR5 HDMI DP DVI (Farbe : Schwarz, Größe : Einheitsgröße)', '669,29\xa0€', 'https://www.amazon.de/KIKIRon-Grafikkarte-6G-Spiel-Videografikkarte-1530MHz-12000MHz/dp/B085RQNRJJ/ref=sr_1_115?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=rx+580+8g&qid=1617643719&sr=8-115')
    ('upHere 5V 3 Pin RGB Grafikkarten Halter, RGB LED Videokarte Sehnenhalter Halterung, GPU Halterung,Motherboard Sync-GS05ARGB', '15,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg8_1?ie=UTF8&adId=A04947471H4Z92BAPDCUZ&url=%2FupHere-Adressierbarer-RGB-Grafikkartenhalter-ARGB-Streifen-einstellbar%25EF%25BC%258CGS05ARGB%2Fdp%2FB082SRSS1P%2Fref%3Dsr_1_116_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643719%26sr%3D8-116-spons%26psc%3D1&qualifier=1617643719&id=8741623108871404&widgetName=sp_mtf')
    ('Corsair Graphite Series 780T PC-Gehäuse (Seitenfenster Full-Tower ATX Performance LED, mit roten LED Lüfter) schwarz', '190,99\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_mtf_aps_sr_pg8_1?ie=UTF8&adId=A00136102ZB9LWAJHI2GH&url=%2FCorsair-PC-Geh%25C3%25A4use-Seitenfenster-Full-Tower-Performance%2Fdp%2FB00LA6POK4%2Fref%3Dsr_1_117_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643719%26sr%3D8-117-spons%26psc%3D1&qualifier=1617643719&id=8741623108871404&widgetName=sp_mtf')
    ('NVIDIA Quadro K600 Grafikkarte (1 GB, DDR3)', '498,00\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg8_1?ie=UTF8&adId=A02655473KNRUJVN4D4PW&url=%2FNVIDIA-Quadro-K600-Grafikkarte-DDR3%2Fdp%2FB07SQFW8X5%2Fref%3Dsr_1_118_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643719%26sr%3D8-118-spons%26psc%3D1&qualifier=1617643719&id=8741623108871404&widgetName=sp_btf')
    ('Corsair Obsidian Series 750D Airflow Edition PC-Gehäuse (Seitenfenster Full Tower ATX High Airflow Performance) schwarz', '147,77\xa0€', 'https://www.amazon.de/gp/slredirect/picassoRedirect.html/ref=pa_sp_btf_aps_sr_pg8_1?ie=UTF8&adId=A00137663B4Q9LFIM12DQ&url=%2FCorsair-Obsidian-PC-Geh%25C3%25A4use-Seitenfenster-Performance%2Fdp%2FB00YJJBFIO%2Fref%3Dsr_1_119_sspa%3F__mk_de_DE%3D%25C3%2585M%25C3%2585%25C5%25BD%25C3%2595%25C3%2591%26dchild%3D1%26keywords%3Drx%2B580%2B8g%26qid%3D1617643719%26sr%3D8-119-spons%26psc%3D1&qualifier=1617643719&id=8741623108871404&widgetName=sp_btf')
    

### Great Job!
