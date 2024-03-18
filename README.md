# DNSChef + CAA patch + ini file to use with certbot 
This is guide on how to issue wildcard certificate for your domain on your own NS server using customized [DNSChef](https://github.com/iphelix/dnschef) with [certbot](https://certbot.eff.org/).

# :lock: Why?
I've stumbled upon this problem when I tried to issue wildcard certificate to use with [interactsh-server](https://github.com/projectdiscovery/interactsh) instance.<br><br>
After some *voodoo googling* I've found no materials that would guide me or even give exact steps to issue wildcard certificate for my domain. So, this is it.

# 🔐 How does it work?
So, the easiest way to issue certificate is using [certbot](https://certbot.eff.org/). It helps you to issue LetsEncrypt certificate.<br><br>
To issue wildcard certificates, you need to prove you have access to your nameserver. It's quite easy to do if you are using default NS given to you by your domain registrar. But not if you have custom NS records pointing to your server.<br><br>
So, two things that your nameserver needs to do is host two DNS records: CAA and TXT. <br>
CAA records "allows" some entities to issue a certificate for your domain.<br>
TXT record is a challenge, given by certbot to prove that you're in control of given nameserver.<br><br>
I've tried to do this with [DNSChef](https://github.com/iphelix/dnschef), but it doesn't support CAA records by default, so I've patched it with the code i found (look for code snippet in Notes section)

# 🔓 Exact steps to issue wildcard certificate
- Clone this repo with `git clone https://github.com/numoworld/dnschef_for_certbot.git`
- Go to downloaded folder with `cd dnschef_for_certbot`
- Install requirements with `pip install -r requirements.txt` or `pip3 install -r requirements.txt` (preferrably inside virtual environment)
- Open `certbot.ini` with any text editor and change all `your.domain` entries with your exact domain  (don't change `_acme-challenge.`)
- Download `certbot` following [this instructions](https://certbot.eff.org/instructions) (use software=other and your system)
- Run certbot with `certbot certonly --manual --preferred-challenge dns -d '*.your.domain'` (substitude `your.domain` with your domain)
- **DO NOT CLOSE TERMINAL WITH CERBOT UNTIL THE VERY END!!!**
- After submitting email and agreeing to some terms, you would stumble upon something that looks like this:
  ```
  Please deploy a DNS TXT record under the name:

  _acme-challenge.your.domain.

  with the following value:

  u669OCD675lyGIc_jufnQsMFeAinKjhiP7bHSLFOAko
  ```
- Replace `your_challenge` string in `certbot.ini` with this value (in my example - `u669OCD675lyGIc_jufnQsMFeAinKjhiP7bHSLFOAko`):
  ```ini
  [CAA]
  your.domain=0 issuewild letsencrypt.org

  [TXT]
  _acme-challenge.your.domain=u669OCD675lyGIc_jufnQsMFeAinKjhiP7bHSLFOAko
  ```
- Save file and run DNSChef with `python3 dnschef.py -i 0.0.0.0 --file certbot.ini`
- Press Enter in terminal with `certbot`
- Wait
- ???
- [Go watch some cats](meow.camera)
# Notes
Code snipped used to modify DNSChef is obtained from [dnschef_updated](https://github.com/irsdl/dnschef_updated) repository ([commit](https://github.com/iphelix/dnschef/commit/4f7e5764e0572aafce020614149218d7e40c5d30)).
```python

                    elif qtype == "CAA":
                        flags, tag, value = fake_record.split(" ")
                        flags = int(flags)

                        # dnslib doesn't like trailing dots
                        if value[-1] == ".": value = value[:-1]

                        # Create CAA record
                        response.add_answer(RR(qname, getattr(QTYPE,qtype), rdata=RDMAP[qtype](flags, tag, value)))
```

