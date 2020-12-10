#!/bin/bash

clear

echo -e "\e[43m\e[1mSourceCode Analysis Script for 19th RoEduNet Conference: Networking in Education and Research - 2020"
echo 
echo "Authors: Ilca Lucian [lucian.ilca@unitbv.ro] && Titus Constantin Balan [balan.titus.unitbv.ro]"
echo
echo "*** Usage: ***"
echo
echo "First Step: ./sas 1 [Will download and install all needed software]"
echo "Second Step: ./sas 2 -> https://github.com/QQuick/SimPyLC -> analysis [Will download and analyse the repository]"
echo "Third Step: ./sas 3 [Get information, such as: vulnerabilities, bugs and project name]"
echo

if [ -z $1 ]
then
  distri="*** Unknown Command"
elif [ -n $1 ]
then
  distri=$1
fi

case $distri in

"1") echo "Installation Script / Step 1"
ls
cd /opt;mkdir icsautomated;cd icsautomated;
apt-get remove docker docker-engine docker.io containerd runc
apt-get update
apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
docker pull sonarqube
docker pull sonar-scanner-cli

for i in {1..1}
    do docker run -d --name sonarqube$i --network=host -p 9000:9000 -d sonarqube
    sleep 1
done

wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.5.0.2216-linux.zip
unzip *.zip
mv sonar-scanner-4.5.0* sonar
sleep 1
cd /opt;cd icsautomated;mkdir -p cli;cd cli;
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.5.0.2216.zip
unzip *.zip
echo "SonarQube / SonarScanner / Docker Installed"
;;

### Option Number 2 # SonarScanner  ####
"2") echo "Select Reposetory to Clone:"
echo ""

read -p "Github URL Reposetory: " repos
read -p "Project Name: " projectname

cd /opt;cd icsautomated;mkdir -p cli;cd cli;cd sonar-scanner-4.5.0.2216/conf;

echo "sonar.host.url=http://localhost:9000" >> sonar-scanner.properties
echo "sonar.projectKey=automated:${projectname}" >> sonar-scanner.properties
echo "sonar.sources=." >> sonar-scanner.properties
echo "sonar.projectName=${projectname}" >> sonar-scanner.properties

cd ..;cd bin;
git clone ${repos}

cd /opt;cd icsautomated;cd cli;cd sonar-scanner-4.5.0.2216/bin;
./sonar-scanner
;;

### Option Number 3 # Get Information About Vulnerabilities and Bugs ###
"3") echo -e "\e[1m\e[44m\e[91mGet Vulnerability Information and Project Names: "

echo "Project Name: "
echo 
output=$(curl -sX GET -u admin:admin http://localhost:9000/api/components/search?qualifiers=TRK)
echo $output | jq '.components | .[] | .name '
echo
echo -e "\e[103m\e[90m### Bugs:### "
echo
output=$(curl -sX GET -u admin:admin http://localhost:9000/api/issues/search?types=BUG)
echo $output | jq '.issues | .[] '
echo
echo -e '\e[41m### Vulnerabilities: ### '
echo ''
output=$(curl -sX GET -u admin:admin http://localhost:9000/api/issues/search?types=VULNERABILITY)
echo $output | jq '.issues | .[] '
echo
;;

cl*) echo "Please use option 1, 2 or 3";;
esac

exit 0

#EOF Script
