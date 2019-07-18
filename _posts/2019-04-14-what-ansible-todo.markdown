---
layout: post
title: "앤서블 설치부터 플레이북까지 다뤄보기"
date: 2019-04-14 12:00:00
description:  # Add post description (optional)
img:  # Add image post (optional)
---

### 열심히 일하다보니 어느 순간 자동화에 대해..

로컬 시스템에 쿠버네티스를 위한 VM을 띄워놓고 install script를 vagrant provision으로 작성해도 되지만 

너무 귀찮은 나머지 ansible을 도입해보기로 결정했다.

아래의 스크립트로 일단 python을 설치한다

 본인은 기준으로 ubutu 18.04 LTS를 사용하고 있으며

 설치 스크립트는 다음과 같다.

```bash
    sudo apt-get -y upgrade &&  \ 
    sudo apt-get -y update  &&  \
    sudo apt-get -y install python3-pip
```

이번에는 vagrant로 10개의 vm을 띄운다음 거기에 간단하게 플레이북을 작성하여 nodejs 앱을 띄워보는걸 목표로 

## vagrant 설치
```
    sudo apt-get install vagrant
```

[이 곳](https://vagrantcloud.com) 에서 베이그란트 VM에 필요한 이미지를 받을 수 있다.

베이그란트는 VM을 프로비저닝하기 위한 도구로서 HashCorp사에서 만든 툴이다. 오늘 다룰 내용은 앤서블이니 

베이그란트는 구글에 검색해서 알아보도록하자

# vagrantfile 작성하기 (우분투 16.04)
```

# -*- mode: ruby -*-
# vi: set ft=ruby :

hosts= {
	"n1" => "192.168.77.1",
	"n2" => "192.168.77.2",
	"n3" => "192.168.77.3",
	"n4" => "192.168.77.4"
    "n5" => "192.168.77.5",
	"n6" => "192.168.77.6",
	"n7" => "192.168.77.7",
	"n8" => "192.168.77.8"
    "n9" => "192.168.77.9"
    "n10" => "192.168.77.10"
}

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  check_guest_additions = false
  functional_vboxsf = false

  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL

  config.vm.box = "bento/ubuntu-16.04"
  hosts.each do |name, ip|
  config.vm.hostname = name
  config.vm.define name do |machine|
   machine.vm.network :private_network, ip: ip
   machine.vm.provider :virtialbox do |v|
    v.cpus = 1
    v.name = name
   end
  end
 end
end
```

그 이후에는

```
    sudo vagrant up
```

으로 VM을 띄운다


파이썬을 설치한 이후에 pip로 ansible을 설치한다.

pip로 ansible을 설치하는 스크립트는 아래와 같다.

```
    pip install ansible
```

ansible의 경우 inventory 라는 것을 지원하는데 일종의 메타데이터를 정의하고 이를 사용하여 자동화 대상을 설정할 수 있다.

vagrant내의 private network상에서 수행될 것이다 계정과 비밀번호 전부 vagrant이니 이 점을 참고하도록 한다.

# 인벤토리 작성 
```bash
# host.ini
[stage]
192.168.10.[1:10] ansible_user=vagrant ansible_ssh_pass=vagrant

```

위의 인벤토리 파일은 host가 192.168.10.1 ~ 192.168.10.10 까지의 호스트의 아래와 같은 변수를 적용하는 것이다.

```
 # install_list.yaml

 - git
 - node
```

그리고 변수 파일을 작성한다.

```
# ansible-playbook.yaml

---
- hosts: stage
become: true
var_files: install_list.yaml
starategy: free
tasks:
- name: update apt list
  shell: "apt-get -y upgrade && apt-get -y update"
- name: change vm hostname
  shell: hostname "node-"$(ifconfig | grep 192.168 | awk '{print $2}')
- name: install items
  shell: "apt install -y "{{ item }}"
  with_items: install_list.yaml
- name: clone express server 
  git:
    repo: git@github.com:seongjumoon/node-example.git
    version: master
    dest: /opt/seongju-node-example
    accept_hostkey: yes
- name: install package.json files
  npm:
   path: /opt/seongju-node-example
 - name: started node js
   shell: npm start
   args:
       chdir: /opt/seongju-node-example
```


이후에 playbook을 실행시킨다음에 ssh로 접속하여 nodejs가 떠있는지 확인해보자.
```
 ansible-playbook -i hosts ansible-playbook.yaml
```

된다 끝


참고자료
https://docs.ansible.com/ansible/latest/user_guide/playbooks_special_topics.html?highlight=playbook
