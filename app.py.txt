# import libraries
import urllib2
import smtplib
import schedule
import time
from datetime import datetime, timedelta
from bs4 import BeautifulSoup
from email.MIMEMultipart import MIMEMultipart
from email.MIMEText import MIMEText

# specify the url
def scrape(location):
    quote_page = 'http://www.marineweather.co.nz/forecasts/' + location
    page = urllib2.urlopen(quote_page)
    soup = BeautifulSoup(page, 'html.parser')
    wind = soup.find('tr', attrs={'class': 'wind'})
    windlist = wind.text.strip().replace(" ", "").splitlines()
    swell = soup.find('tr', attrs={'class': 'swell'})
    swelllist = swell.text.strip().replace(" ", "").splitlines()
    thelist = zip(windlist[3:],swelllist)

    now = datetime.now().replace(hour=6, minute=0, second=0)

    body = location + '\n'
    i = 0
    for x in thelist:
        # print now.strftime('%Y-%m-%d %H:%M:%S ') + '   Wind: '+ x[0] + '\n                      Swell: ' + x[1]
        body = body + now.strftime('%Y-%m-%d %H:%M:%S ') + '   Wind: '+ x[0] + ' Swell: ' + x[1]
        i = i + 1
        if i%3 == 0:
            now = now + timedelta(hours=12)
        else:
            now = now + timedelta(hours=6)

    email(body)

def email(body_text):
    fromaddr = ''
    toaddr = ''
    msg = MIMEMultipart()
    msg['From'] = fromaddr
    msg['To'] = toaddr
    msg['Subject'] = 'Daily Swell Report ' + datetime.now().strftime('%Y-%m-%d')

    body = body_text
    msg.attach(MIMEText(body, 'plain'))

    server = smtplib.SMTP('smtp.gmail.com', 587)
    server.starttls()
    server.login(fromaddr, '')
    text = msg.as_string()
    server.sendmail(fromaddr, toaddr, text)
    server.quit()

schedule.every(24).hours.do(scrape('eastbourne'))
schedule.every(24).hours.do(scrape('breaker-bay'))
schedule.every(24).hours.do(scrape('baring-head'))

while 1:
    schedule.run_pending()
    time.sleep(1)
