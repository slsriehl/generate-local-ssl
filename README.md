# Configure DNSMasq and generate a local SSL certificate for Mac

## Install Homebrew
* Open Terminal and run the following:

```
mkdir homebrew && curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew
```

## Install & Configure DNSMasq
* From Terminal, run the following (from https://github.com/wingsuitist/brewamp/blob/master/src/dnsmasq.sh):

```
if (brew services list|grep dnsmasq); then
  tell "DNS Masq guids us to the right IP which is 127.0.0.1 - but it's already here"
else
  tell "DNS Masq guids us to the right IP which is 127.0.0.1 - so lets summon this last deamon"

  brew install -v dnsmasq
  echo 'address=/.dev/127.0.0.1' > $(brew --prefix)/etc/dnsmasq.conf
  echo 'listen-address=127.0.0.1' >> $(brew --prefix)/etc/dnsmasq.conf
  echo 'port=35353' >> $(brew --prefix)/etc/dnsmasq.conf
  brew services start dnsmasq
fi

if [ -e /etc/resolver/dev ]; then
  tell "DNS Masq is already in our resolver list"
else
  tell "So let's call DNS Masq all the time - cause, yes - it's nice"
  sudo mkdir -v /etc/resolver
  sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/dev'
  sudo bash -c 'echo "port 35353" >> /etc/resolver/dev'
fi
```

## Generate the SSL Certificate
from https://gist.github.com/jed/6147872

* Open Terminal and cd into the root folder of your project.

* Create a temporary configuration file:

```
cat > openssl.cnf <<-EOF
  [req]
  distinguished_name = req_distinguished_name
  x509_extensions = v3_req
  prompt = no
  [req_distinguished_name]
  CN = *.${PWD##*/}.dev
  [v3_req]
  keyUsage = keyEncipherment, dataEncipherment
  extendedKeyUsage = serverAuth
  subjectAltName = @alt_names
  [alt_names]
  DNS.1 = *.${PWD##*/}.dev
  DNS.2 = ${PWD##*/}.dev
EOF
```

*  Create the certificate:

```
openssl req \
  -new \
  -newkey rsa:2048 \
  -sha256 \
  -days 3650 \
  -nodes \
  -x509 \
  -keyout ssl.key \
  -out ssl.crt \
  -config openssl.cnf
```

* if you get the error "unable to write to random state", do

```
sudo rm ~/.rnd
```

and create the certificate again.

* Remove the configuration file:

```
rm openssl.cnf
```

* Open the SSL certificate in your keychain:

```
open /Applications/Utilities/Keychain\ Access.app ssl.crt
```

* Select the newly imported certificate, which should appear at the bottom of the certificate list, right click, and select "Get Info".

* In the popup window, click the ▶ button to the left of Trust, and select Always Trust for When using this certificate:.

* Close the popup window.

* When prompted, enter your password again and click Update Settings.

* Close Keychain Access.
