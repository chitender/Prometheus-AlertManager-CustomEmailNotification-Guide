# Prometheus-AlertManager-CustomEmailNotification-Guide
step by step guide for enabling custom email notifications from Prometheus/Alertmanager
# to send custom email notification
This documents shows step by step guide how we can send the custom email notification.

Below example is to customise the email notification to include specific labels( Description, Project, Environment, Instance) , Annotations, custom email subject  and a Grafana dashboard link.


## Steps

1. below is the sample prometheus rule which checks for disk utilisation.
  - alert: Disk Used
    expr: 100 * (1 - (node_filesystem_free_bytes / node_filesystem_size_bytes)) > 60
    for: 5m
    labels:
      severity: Warning
      subject: 'Disk utilization is {{ $value | humanize }}%  for {{ $labels.mountpoint }} on {{ $labels.Name }} in {{ $labels.Project }}-{{ $labels.Environment }}'
    annotations:
      summary: "Disk Usage is High"
      identifier: '{{ $labels.instance }}'
      description: 'The Disk Utilization for Disk {{ $labels.mountpoint }} is high on {{ $labels.instance }} in project {{ $labels.Project }} is {{ $value | humanize }} percent on {{ $labels.Environment }} environment'
      graph: 'https://monitoring.organisation.com/grafana/d/O6JPd-DWk/node-exporter-full?orgId=1&var-Project={{ $labels.Project }}&var-job={{ $labels.job}} &var-name={{ $labels.nodename }}&var-node={{ $labels.nodename }}&var-port=9100&fullscreen&panelId=152'

2. prepare your custom Html template file.
    here custom_email.tmpl is as an example, where we only included the specific labels( Description, Project, Environment, Instance) and Annotations in the email notifications.
    custom_email.tmpl [here] (custom_email.tmpl)

    at the top in custom_email.tmpl we haved defined the template name ( {{ define "email.CUSTOM.html" }}) which we will use in the alertmanager config file.

3. now we will include this custom email template in our alertmanager config so that Alertmanager will recognise this          template. for this we will use template module. like below example:
    templates:
    - '/prometheus/alertmanager/etc/alertmanager/templates/custom_email.tmpl'

4. now we will add the receiver in our alertmanager.yml to who we want to send custom email notifications. below is a          sample config.
    - name: Devops-test
      email_configs:
      - to: chitenderkumar.16@gmail.com
        from: chitender.kumar@delhivery.com (Prometheus-Dev-Alerts)
        smarthost: smtp.gmail.com:587
        auth_identity: chitender.kumar@delhivery.com
        auth_username: chitender.kumar@delhivery.com
        auth_password: fhkfkfltmmgwtsuh
        headers:
          Subject: ' {{ .GroupLabels.subject }} '
        html: '{{ template "email.CUSTOM.html" . }}'

5. once we are done with above changes. our alertmanager.yml will look something like below:
    global:
    route:
      group_by: ['alertname', 'Project','instance', 'severity', 'Environment', 'Name', 'subject', 'job', 'nodename']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 30m
      receiver: Devops-test
    receivers:
    - name: Devops-test
      email_configs:
      - to: chitenderkumar.16@gmail.com
        from: chitender.kumar@delhivery.com (Prometheus-Dev-Alerts)
        smarthost: smtp.gmail.com:587
        auth_identity: chitender.kumar@delhivery.com
        auth_username: chitender.kumar@delhivery.com
        auth_password: fhkfkfltmmgwtsuh
        headers:
          Subject: ' {{ .GroupLabels.subject }} '
        html: '{{ template "email.chitender.html" . }}'
    templates:
    - '/prometheus/alertmanager/etc/alertmanager/templates/custom_email.tmpl'

6. now reload your alertmanager. i do like below:
    curl -X POST http://localhost:9093/-/reload


## Usage

with above custom email template, your email notification will looks like something below:
![alt text] (custom_email_notifications.png)
