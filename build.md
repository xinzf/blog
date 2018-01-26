## golang
```make
version = ${BUILD_NUMBER}
name = ${JOB_NAME}
cluster = ${CLUSTER}


.PHONY: build
build:
	mkdir ${GOSRC}/pipifit/$(name)/build -p;
	cd ${GOSRC}/pipifit/$(name) && glide install && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags '-w' -i -o ${GOSRC}/pipifit/$(name)/build/$(version)

deploy:
ifeq ($(cluster), dev)
	cd ${GOSRC}/pipifit/$(name) && make -f ${JENKINS_HOME}/build-makefiles/golang beta
else
ifeq (${BUILD_USER_ID}, admin)
	cd ${GOSRC}/pipifit/$(name) && make -f ${JENKINS_HOME}/build-makefiles/golang stable
else
	echo "你没有权限"
	exit 254
endif
endif

beta:
	mkdir ${GO_PROJECT_ROOT}/$(name)/current -p && mkdir ${GO_PROJECT_ROOT}/$(name)/releases -p;
	cp ${GOSRC}/pipifit/$(name)/build/${BUILD_NUMBER} ${GO_PROJECT_ROOT}/$(name)/releases/;
	rm -rf ${GO_PROJECT_ROOT}/$(name)/current/$(name);
	ln -s ${GO_PROJECT_ROOT}/$(name)/releases/${BUILD_NUMBER} ${GO_PROJECT_ROOT}/$(name)/current/$(name);
	supervisorctl -c ~/.supervisor/supervisor.ini restart $(name)

stable:
ifeq ($(cluster), micro)
	for i in micro1-go; do \
		ssh -t $$i 'mkdir /var/go_projects/$(name)/current -p && mkdir /var/go_projects/$(name)/releases -p' && \
		scp ${GOSRC}/pipifit/$(name)/build/${BUILD_NUMBER} $$i:/var/go_projects/$(name)/releases/ && \
		ssh -t $$i 'rm -rf /var/go_projects/$(name)/current/$(name) && ln -s /var/go_projects/$(name)/releases/$(version) /var/go_projects/$(name)/current/$(name)'  && \
		ssh -t $$i 'supervisorctl -c ~/.supervisor/supervisor.ini restart $(name)'; \
	done
endif
```
## php
```make
version = ${BUILD_NUMBER}
name = ${JOB_NAME}
cluster = ${CLUSTER}


.PHONY: build
build:
	mkdir ${PHPSRC}/$(name)/build -p;
	cd ${PHPSRC}/$(name)/src && ${JENKINS_HOME}/build-makefiles/phplint/bin/phplint --configuration=${JENKINS_HOME}/build-makefiles/phplint/config.yml
	cd ${PHPSRC}/$(name)/src && /usr/local/bin/composer update;
	cd ${PHPSRC}/$(name)/src && tar -czvf ${PHPSRC}/$(name)/build/$(version).tar.gz . --exclude=*.svn --exclude=*.git --exclude=*.repo --exclude=composer.json --exclude=composer.lock

deploy:
ifeq ($(cluster), dev)
	cd ${PHPSRC}/$(name) && make -f ${JENKINS_HOME}/build-makefiles/php beta
else
ifeq (${BUILD_USER_ID}, admin)
	cd ${PHPSRC}/$(name) && make -f ${JENKINS_HOME}/build-makefiles/php stable
else
	echo "你没有权限"
	exit 254
endif
endif

beta:
	mkdir ${PHP_PROJECT_ROOT}/$(name)/current -p && mkdir ${PHP_PROJECT_ROOT}/$(name)/releases/${BUILD_NUMBER} -p;
	tar -zxvf ${PHPSRC}/$(name)/build/${BUILD_NUMBER}.tar.gz -C ${PHP_PROJECT_ROOT}/$(name)/releases/${BUILD_NUMBER} --strip-components 1;
	rm -rf ${PHP_PROJECT_ROOT}/$(name)/current;
	ln -s ${PHP_PROJECT_ROOT}/$(name)/releases/${BUILD_NUMBER} ${PHP_PROJECT_ROOT}/$(name)/current;
	supervisorctl -c ~/.supervisor/supervisor.ini restart php

stable:
ifeq ($(cluster), micro)
	for i in micro1-www; do \
		ssh -t $$i 'mkdir /var/www/$(name)/current -p && mkdir /var/www/$(name)/releases/$(version) -p' && \
		scp ${PHPSRC}/$(name)/build/${BUILD_NUMBER}.tar.gz $$i:/var/www/$(name)/releases/ && \
		ssh -t $$i 'tar -zxvf /var/www/$(name)/releases/$(version).tar.gz -C /var/www/$(name)/releases/$(version) --strip-components 1 && rm -rf /var/www/$(name)/current && ln -s /var/www/$(name)/releases/$(version) /var/www/$(name)/current'  && \
		ssh -t $$i 'supervisorctl -c ~/.supervisor/supervisor.ini restart php-fpm'; \
	done
endif

ifeq ($(cluster), admin)
	for i in admin1-www; do \
		ssh -t $$i 'mkdir /var/www/$(name)/current -p && mkdir /var/www/$(name)/releases/$(version) -p' && \
		scp ${PHPSRC}/$(name)/build/${BUILD_NUMBER}.tar.gz $$i:/var/www/$(name)/releases/ && \
		ssh -t $$i 'tar -zxvf /var/www/$(name)/releases/$(version).tar.gz -C /var/www/$(name)/releases/$(version) --strip-components 1 && rm -rf /var/www/$(name)/current && ln -s /var/www/$(name)/releases/$(version) /var/www/$(name)/current'  && \
		ssh -t $$i 'supervisorctl -c ~/.supervisor/supervisor.ini restart php-fpm'; \
	done
endif

ifeq ($(cluster), api)
	for i in api1-www; do \
		ssh -t $$i 'mkdir /var/www/$(name)/current -p && mkdir /var/www/$(name)/releases/$(version) -p' && \
		scp ${PHPSRC}/$(name)/build/${BUILD_NUMBER}.tar.gz $$i:/var/www/$(name)/releases/ && \
		ssh -t $$i 'tar -zxvf /var/www/$(name)/releases/$(version).tar.gz -C /var/www/$(name)/releases/$(version) --strip-components 1 && rm -rf /var/www/$(name)/current && ln -s /var/www/$(name)/releases/$(version) /var/www/$(name)/current'  && \
		ssh -t $$i 'supervisorctl -c ~/.supervisor/supervisor.ini restart php-fpm'; \
	done
endif
```
