```
###########################################################################
# XHPROF:
###########################################################################

ARG INSTALL_XHPROF=true
RUN if [ ${INSTALL_XHPROF} = true ]; then \
  git clone https://github.com/longxinH/xhprof.git ./xhprof
  cd xhprof/extension/ && \
  /usr/local/bin/php/phpize && \
  ./configure --with-php-config=/usr/local/bin/php/php-config && \
  make && sudo make install && \
  rm -f xhprof && \
  docker-php-ext-install xhprof \
;fi
```
