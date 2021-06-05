Access policy 
-This program does not reside on the public internet. This service is exclusive to the company and the people associated with the company. This also requires the teammate to use the same account that the request was made for. The elk server (azure) will not work with computers that are not register with the azure services.

-this service is under the ip address(   ) . To probably access this service you will need to use the public ip address for the services.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Usage  
-Kibana is the program that resides on the elk server and its main purpose is to collect data on network traffic. This allows us to be aware of the quality of performance that takes place on the servers and network within the company. Understanding and collecting this data allows use to maintain great service to the visitors of the website and also maintain a security environment on the network. This setting and data being tracked is set up by the network admin.Some of the information collected by our team is network traffic, visitor ip address location and geo coordinates of users.  

-The kibana services is a web application that will reside on the elk server.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Setup process

A new virtal network need to be made.
Then create a peer network. This will connect rednet and the network you just continued. 
Then you want to make a virtal network.(call it elkVM)
Then I added the container and then went to the host file.
Then I added the ip address to the host file under webserver section.
Then I create a playbook then I added the applicable code. (check below to see speciic code)
Then I executed he scrpited.
Then I check to see if the kibana was working correctly.
Then I added addtional scripted to the playbook to add filebeat.
Then I excuted the scripted for filebeat.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Program usage discovers

Within the usage of this program we have learned quite a few details regarding activities currently going on the network. Understanding the network activity can allow for us to provide a better service to our clientele and mitigate potential security threats. The following section will cover our initial findings.

We had 253 unique visitors from India over the last 7 days.

Within the last 24 hours 7 users from china used macos.
Over the last week most of the residents from china create the majority of the traffic. The majority of the traffic from china was done between 12 pm - 1 pm.

The most download file types over the last week was .gz( This file type is used for compression. This uses the gzip compression utility), .css( This is relate to html webpage ), .zip( a file format used for compression) , .deb (a file format used for debian linux app installs ) and .rpm (this is a red hat software package file. )

There was a user found to be using more bytes then other users for the same activity. This event was time stamped and was found to have taken place on september 13, 2020. The visitor downloaded a RPM file and its origin location was from india. The visitor received a http response code of 200, which indicates that the response was successful. The source  ip address was 35.143.166.159 and it geo coordinates was  
{ "lat": 43.34121, "lon": -73.6103075 }. The operating system the visitor was using was windows 8 and accessed url was
https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-6.3.2-i686.rpm. The traffic originated from the facebook website. We concluded that this incident was a user downloading a linux package from the website being monitored. The referral link from facebook initially gave some concern but after analysis it was decided not a threat. We would encourage to notify the user that post package update links on Facebook goes against our compliance policy. This user may be monitored in the future. 
 
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
extra details

 - name: Setup Elk Server
    hosts: elk
    become: true
    tasks:
    - name: set vm.max_map_count to 262144 in sysctl
      sysctl: name={{ item.key }} value={{ item.value }}
      with_items:
      - { key: "vm.max_map_count", value: "262144" }
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
    - name: Install Python Docker Module
      pip:
        name: docker
        state: present
    - name: Download and Launch the sebp elk 761 Docker
      docker_container:
        name: sebp
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports: 5601:5601,9200:9200,5044:5044
    - name: Enable Docker Service
      systemd:
        name: docker
        enabled: yes
---------------------------------------------------------------------------------------------------------------------------------------------------------------

 - name: installing and launching filebeat
   hosts: webservers
   become: yes
   tasks:
   - name: download filebeat deb
     command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb
   - name: install filebeat deb
     command: dpkg -i filebeat-7.6.1-amd64.deb
   - name: drop in filebeat.yml
     copy:
       src: /etc/ansible/files/filebeat-config.yml
       dest: /etc/filebeat/filebeat.yml
       mode: 0644
   - name: enable and configure system module
     command: filebeat modules enable system
   - name: setup filebeat
     command: filebeat setup
   - name: start filebeat service
     command: service filebeat start
   - name: enable service filebeat on boot
     systemd:
       name: filebeat
       enabled: yes


