# Add a new user

adduser user
cd /home/user
cp -r ~/.ssh .
chown -R user .ssh

# Firewall - disable all expect SSH

ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw enable
ufw status

# Install Docker

apt-get remove docker docker-engine docker.io
apt-get update
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update
apt-get -y install docker-ce

docker run hello-world

# Prepare for build

mkdir ~/m2
mkdir ~/deploy

chmod 777 m2 deploy

# Start deploy container

docker run -it --name mhus-deploy \
 -h deploy-mhus \
 -v ~/m2:/home/user/.m2 \
 -v ~/deploy:/home/user/deploy \
 -v /var/run/docker.sock:/var/run/docker.sock \
 --privileged=true \
 mhus/mhus-deploy:11.2
 
# Inside the deploy container, build all
 
cd deploy
git clone https://github.com/mhus/mhus-parent.git
cd mhus-parent
con git.pull
con rebuild
cd ..

# Execute this outside of the container to enable access to the docker socket

docker exec --user=0 mhus-deploy chmod 777 /var/run/docker.sock

# Start unit tests inside of the contianer

git clone https://github.com/mhus/mhus-itests.git
cd mhus-itests
export KARAF_M2_HOME=/root/m2
mvn install


