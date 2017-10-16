# Configure DNSMasq and generate a local SSL certificate for Mac

## Install Homebrew
1. Open Terminal and run the following:

```
mkdir homebrew && curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew
```

## Install & Configure DNSMasq
1. Run the following (from https://github.com/wingsuitist/brewamp/blob/master/src/dnsmasq.sh):

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

1. Open the command line and cd into the root folder of your project.

1. Create a temporary configuration file:

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

1.  Create the certificate:

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

1. if you get the error "unable to write to random state", do

```
sudo rm ~/.rnd
```

and create the certificate again.

1. Remove the configuration file:

```
rm openssl.cnf
```

1. Open the SSL certificate in your keychain:

```
open /Applications/Utilities/Keychain\ Access.app ssl.crt
```

1. Select the newly imported certificate, which should appear at the bottom of the certificate list, right click, and select "Get Info".

1. In the popup window, click the ▶ button to the left of Trust, and select Always Trust for When using this certificate:.

1. Close the popup window.

1. When prompted, enter your password again and click Update Settings.

1. Close Keychain Access.
