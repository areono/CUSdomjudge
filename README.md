# 1. 설치 가이드

```shell
    
    # **Domjudge Docker**
    
    설치는 **리눅스 환경**을 기본으로 합니다.
    
    # **파일 다운로드**
    
    일단, [도커](https://www.docker.com/) 와 [도커 컴포즈](https://docs.docker.com/compose/)가 깔려 있어야 합니다.
    
    깔려 있지 않다면 [도커 설치](https://docs.docker.com/compose/install/), [도커 컴포즈 설치](https://docs.docker.com/compose/install/) 공식 홈페이지 내용을 보고 설치 합니다.
    
    설치가 완료되면 자신이 원하는 경로에 파일을 제작합니다
    
```
    $ sudo vi docker-compose.yml
```
    version: '3.9'
    
    networks:
      domjudge:
        name: domjudge
    
    services:
      mariadb:
        container_name: mariadb
        image: mariadb:latest
        networks:
          - domjudge
        ports:
          - 13306:3306
        environment:
          - MYSQL_ROOT_PASSWORD=rootpw
          - MYSQL_USER=domjudge
          - MYSQL_PASSWORD=djpw
          - MYSQL_DATABASE=domjudge
        command: --max-connections=1000
    
      domserver:
        container_name: domserver
        image: domjudge/domserver:latest
        volumes:
          - /sys/fs/cgroup:/sys/fs/cgroup  # 만약 cgroup read-only 에러가 발생한다면 이부분의 :ro가 있는지 확인
        networks:
          - domjudge
        ports:
          - 12345:80
        depends_on:
          - mariadb
        environment:
          - CONTAINER_TIMEZONE=Asia/Seoul
          - MYSQL_HOST=mariadb
          - MYSQL_ROOT_PASSWORD=rootpw
          - MYSQL_USER=domjudge
          - MYSQL_PASSWORD=djpw
          - MYSQL_DATABASE=domjudge
    
      judgehost-0:
        container_name: judgehost-0
        image: domjudge/judgehost:latest
        privileged: true
        hostname: judgedaemon-0
        volumes:
          - /sys/fs/cgroup:/sys/fs/cgroup  # 만약 cgroup read-only 에러가 발생한다면 이부분의 :ro가 있는지 확인
        networks:
          - domjudge
        depends_on:
          - domserver
        environment:
          - CONTAINER_TIMEZONE=Asia/Seoul
          - JUDGEDAEMON_USERNAME=judgehost
          - DAEMON_ID=0
          - JUDGEDAEMON_PASSWORD = <judgehost password>
```
로컬로 진행시 [http://loaclhost:12345](http://loaclhost:12345/) 로 확인가능
    
# **judgehsot password 찾기**
    
```
    $ docker compose up -d domserver mariadb
    
    $ docker exec -it domserver cat /opt/domjudge/domserver/etc/restapi.secret
    
    $ default	http://local/domjudge/api	judgehost <judgehost pasword>
```
# **실행**
    

    $ docker compose up -d <judgehostname>
    
### **judgehost의 이름이 기억나지 않는다면**

```
    $ docker ps -a
```

### **관리자 계정 초기비번확인**
    
```
    $ docker exec -it domserver cat /opt/domjudge/domserver/etc/initial_admin_password.secret
    
    <admin password>
```
    
# **로그인 확인**
    
```
    초기비번확인 후
    ID : admin
    pw : <admin password>
```
    

# 2. 문제 및 해결

- 에러사진 ssh
    
    ![ssh4.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e0045713-8254-4a99-8133-738edc1c294c/6f0d36a1-10ac-4f72-95d5-67e3f0a99f81/ssh4.png)
    
    ![ssh1.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e0045713-8254-4a99-8133-738edc1c294c/54b136b8-556a-4e29-acfd-e265997addfd/ssh1.png)
    
    ![ssh2.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e0045713-8254-4a99-8133-738edc1c294c/893f6fed-ef03-4e5b-84f2-a001aea5386d/ssh2.png)
    
    ![ssh3.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e0045713-8254-4a99-8133-738edc1c294c/82a7b80d-2945-4a2e-bd69-191b172fe049/ssh3.png)
    

- 성공사진 로컬
    
    ![local4.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e0045713-8254-4a99-8133-738edc1c294c/237d7124-1f8e-44a1-9b84-e2c1fc1ca18e/local4.png)
    
    ![local1.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e0045713-8254-4a99-8133-738edc1c294c/12bf67df-0aba-4b41-abfb-b97778682d6a/local1.png)
    
    ![local2.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e0045713-8254-4a99-8133-738edc1c294c/f381a7bd-17e7-47e4-b2b7-2d4e6e29c14b/local2.png)
    
    ![local3.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e0045713-8254-4a99-8133-738edc1c294c/72b61e94-1863-43ae-a566-ccae4ad8b28d/local3.png)
    

<aside>
💡

발생문제 및 해결과정

- [x]  domserver와 db의 컨테이너는 전부 올렸지만 judgehost를 찾지 못했던것 → domjudge의 judgehost를 찾아 컨테이너에 추가
- [x]  cgurop오류 발생 → domjudge에서 알려주는 /etc/default/grub 의 경로에  아래의 설정을 추가

```jsx
GRUB_CMDLINE_LINUX_DEFAULT="quiet cgroup_enable=memory swapaccount=1 systemd.unified_cgroup_hierarchy=0"
```

- [x]  domserver 과 judgehost가 연결이 안됨 → docker-compose.yml에 경로 설정 파일이 read-only 파일임을 찾아 수정
- [x]  judgehsot의 인증 과정에서 오류가 발생 → docker-compose.yml에 judgehost의 비밀번호 부분이 domserver의 restapi.secret에 있을 발견하여 비빌번호 수정으로  해결
- [x]  문제 제출 과정에서 400page 오류 발생 → 출제된 문제에 testcase가 없어 발생한 오류임을 확인

 

</aside>

- 24-09-10
    - 위의 경로에 해당하는 파일에서 role 추가 가능해보임
    - https://github.com/DOMjudge/domjudge/blob/main/webapp/src/DataFixtures/DefaultData/RoleFixture.php
- 24-09-09
    - 240907~240908 >> domserver안에서 judgehost로 연결하는 경로에서 문제가 발생되는 에러로그를 확인 domserver 컨테이너 안에  [localhost](http://localhost) 경로를 서버 도메인으로 변경  후 …
    - 240905~240906 >> judgehost 설치 후 docker에서 컨테이너가 계속해서 다운되는 문제점을 발견, 교수님과 에러 로그를 확인 후 권한 문제임을 알게 되었고 docker-compose.yml 파일에서 시스템에 관여되는 파일 경로가 :ro(readonly) 로 설정 되어있음을 확인 수정 후 재 접속 여전히 다운되는 문제점을 발견
    - 240904 >> test문제를 올린 후 문제 제출 과정 중 문제 채점을 위한 judgehost가 필요하다는 정보를 찾은 후 domjudge사이트에서 제공하는 judgehost를 사용
    - 240902~240904 >> domjudge사이트에서 제공하는 docker image를 이용하여 domserver 과 domjudge에서 사용하는 mariadb를 연결하는데 성공
    
    ### 24-09-11
    
    ```bash
    if you both mariadb and domserver compose up docker container occured error you can tried //분리 해서 해보세요
    ```
    
    start
    
    ```jsx
    $ docker compose up -d mariadb 
    $ docker logs mariadb
    .....
    $ docker compose up -d domserver
    $ docker logs domserver
    ..... check the complete upload....
    modify docker-compose.yml in judgehost-0 judge_password
    find this line
    $ docker exec -it domserver cat /opt/domjudge/domserver/etc/restapi.secret
    .... and
    $ docker compose up -d
     and check this message
     useradd warning: domjudge-run-0's uid 62860 outside of the UID_MIN 1000 and UID_MAX 60000 range.
    [Sep 11 01:57:06.619] judgedaemon[195]: Judge started on judgedaemon-0-0 [DOMjudge/8.2.3]
    [Sep 11 01:57:06.619] judgedaemon[195]: 🔏 Executing chroot script: 'chroot-startstop.sh check'
    [Sep 11 01:57:06.632] judgedaemon[195]: Registering judgehost on endpoint default: http://domserver/api/v4
    [Sep 11 01:57:07.132] judgedaemon[195]: No submissions in queue (for endpoint default), waiting...
    ```
