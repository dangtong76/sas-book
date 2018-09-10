##SAS viya 3.3 to 3.4 upgrade Guide

### 디렉토리 구조

| 디렉토리            | 설명                       |
| ------------------- | -------------------------- |
| /opt/install        | 3.3 인스톨 수행 디렉토리   |
| /opt/upgrade        | 3.4 인스톨 수행 디렉토리   |
| /opt/upgrade/mirror | 3.4 인스톨 미러 리포지토리 |

### vars.yml 파일 복사

#### 수정되지 않은 3.3 vars.yml 파일 복사

~~~{bash}
cp /opt/install/sas_viya_playbook/samples/vars.yml /opt/upgrade/sas_viya_playbook/vars_original.yml
~~~

#### 3.3 인스톨시 수정된 vars.yml 파일 복사

~~~{bash}
cp /opt/install/sas_viya_playbook/vars.yml /opt/upgrade/sas_viya_playbook/vars_current.yml
~~~

#### 파일비교

~~~{bash}
cd /opt/upgrade/sas_viya_playbook
diff vars_current.yml vars.yml
~~~



~~~bash
ansible all -m shell --become --become-user=root -a 'yum remove --assumeyes $(rpm -qf --qf "::%{group}::%{name}\n" /etc/yum.repos.d/*.repo | sed -e "/^::SAS::/!d" -e "s/^::SAS:://" | sort -u)'
~~~

