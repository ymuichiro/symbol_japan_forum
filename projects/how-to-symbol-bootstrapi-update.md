sudo su user

# backup
cd ~
cp ~/target/nodes/node/data/harvesters.dat ~/backup/harvesters.dat
cp ~/target/preset.yml ~/backup/preset.yml
cp ~/target/addresses.yml ~/backup/addresses.yml

# Update
npm install -g symbol-bootstrap@1.1.x

# Reload
symbol-bootstrap stop
symbol-bootstrap config -p mainnet -a dual -c custom-preset.yml --upgrade
symbol-bootstrap compose --upgrade
symbol-bootstrap run -d

# Check
symbol-bootstrap healthCheck
