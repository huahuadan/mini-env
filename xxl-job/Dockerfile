FROM eclipse-temurin:8-jre

# 设置工作目录
WORKDIR /app

# 复制 XXL-JOB 的 JAR 包到容器中
COPY xxl-job-admin-2.3.1.jar /app/xxl-job-admin.jar

# 暴露端口
EXPOSE 8080

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 启动 XXL-JOB，使用外部配置文件
ENTRYPOINT ["java", "-jar", "/app/xxl-job-admin.jar", "--spring.config.location=file:/app/config/application.properties"]

