# ServiceFunctionChaining

ONOS OpenStack 설치 및 통합방

1. ONOS 설치
1.1 ONOS 설치
  git clone https://gerrit.onosproject.org/onos
  cd onos
  git checkout master(or onos-version)
  
1.2 Java 설치
  sudo apt-get install software-properties-common -y
  sudo add-apt-repository ppa:webupd8team/java -y
  sudo apt-get update
  sudo apt-get install oracle-java8-installer oracle-java8-set-default -y
  export JAVA_HOME=/usr/lib/jvm/java-8-oracle

1.3 Maven, Karaf 설치
  cd; mkdir Downloads Applications
  cd Downloads
  wget http://archive.apache.org/dist/karaf/3.0.5/apache-karaf-3.0.5.tar.gz
  wget http://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
  tar -zxvf apache-karaf-3.0.5.tar.gz -C ../Applications/
  tar -zxvf apache-maven-3.3.9-bin.tar.gz –C ../Applications/
  
1.4 ONOS 환경설정
  vi ~/.bash_profile
  export ONOS_ROOT=~/onos
  source $ONOS_ROOT/tools/dev/bash_profile
  export ONOS_IP=<ONOS Controller IP>
  
1.5 ONOS 빌드 및 실행
  . ~/.bash_profile

  cd ~/onos
  mvn clean install
  ok clean
  
2. OpenStack 설치
2.1 networking-sfc 설치
  sudo mkdir /opt/stack
  cd /opt/stack/
  sudo git clone git://git.openstack.org/openstack/networking-sfc.git -b stable/mitaka

2.2 devstack 설치
  git clone https://git.openstack.org/openstack-dev/devstack -b stable/mitaka

2.3 local.conf 파일 생성
  vi local.conf
  [[local|localrc]]
  DEST=/opt/stack 
  #OFFLINE=True 

  # Logging 
  LOGFILE=$DEST/logs/stack.sh.log 
  VERBOSE=True 
  LOG_COLOR=False 
  SCREEN_LOGDIR=$DEST/logs/screen 

  # Credentials 
  ADMIN_PASSWORD=openstack 
  MYSQL_PASSWORD=openstack 
  RABBIT_PASSWORD=openstack 
  SERVICE_PASSWORD=openstack 
  SERVICE_TOKEN=tokentoken 

  # Neutron - Networking Service 
  #DISABLED_SERVICES=n-net 
  ENABLED_SERVICES+=,q-svc,q-agt,q-dhcp,q-l3,q-meta,neutron
  enable_plugin networking-sfc /opt/stack/networking-sfc 

2.4 devstack 실행
  ./stack.sh

3. ONOS, OpenStack 통합
3.1 커널 업데이트, ovs, nsh 설치
  sudo apt-get install linux-image-generic-lts-wily linux-headers-generic-lts-wily
  sudo reboot
  sudo apt-get install openvswitch-switch
  ./install_ovs_nsh.sh
  
3.2 networking-onos 설치
  git clone https://github.com/openstack/networking-onos.git
  cd networking-onos
  sudo python setup.py install
  
3.3 /etc/neutron/plugins/ml2/conf_onos.ini 생성
  vi conf_onos.ini
  [onos]
  url_path = http://<ONOS Controller IP>:8181/onos/vtn
  username = karaf 
  password = karaf
  
3.4 /etc/neutron/plugins/ml2/ml2.conf 파일 수정
  ···
  mechanism_drivers = onos_ml2
  ···
  
3.5 /opt/stack/neutron/neutron.egg-info/entry_points.txt 파일 수정
  ···
  [neutron.ml2.mechanism_drivers]
  ···
  onos_ml2 = networking_onos.plugins.ml2.driver:ONOSMechanismDriver
  ···
  [neutron.service_plugins]
  ···
  onos_router = networking_onos.plugins.l3.driver:ONOSL3Plugin
  ···
  
3.6 neutron-server 재시작
  screen -x –r stack
  q-svc에 n-cpu까지 재시

3.7 ONOS feature 설치
  feature:install onos-openflow
  feature:install onos-openflow-base
  feature:install onos-ovsdatabase
  feature:install onos-ovsdb-base
  feature:install onos-drivers-ovsdb
  feature:install onos-ovsdb-provider-host
  feature:install onos-app-vtn-onosfw
  externalportname-set –n onos_port2

3.8 ovs manager 설정
  sudo ovs-vsctl set-manager tcp:<ONOS IP>:6640
