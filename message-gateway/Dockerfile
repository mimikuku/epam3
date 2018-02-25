FROM maven:3.5-jdk-8-onbuild 
COPY . /home/root/message-gateway 
WORKDIR /home/root/message-gateway 
ENTRYPOINT ["mvn"] 
CMD ["tomcat7:run"]

