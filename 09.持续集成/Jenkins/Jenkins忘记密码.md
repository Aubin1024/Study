 - 修改配置中的密码
`/var/lib/jenkins/`为jenkins的根目录
```
/var/lib/jenkins/users/root/config.xml
<passwordHash>#jbcrypt:$2a$10$e2W9YA1t1UX358CxVAw.kORLrior2EO2ven8TE0dqMpg/AnRlw3LC</passwordHash>
```

 - 替换hash值  
替换后密码为111111
```
<passwordHash>#jbcrypt:$2a$10$DdaWzN64JgUtLdvxWIflcuQu2fgrrMSAMabF5TSrGK5nXitqK9ZMS</passwordHash>
```
