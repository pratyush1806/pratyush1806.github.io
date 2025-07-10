---
layout: post
title: "Monitoring Node.js Applications Using PM2"
author: pratyush
categories: [ Tutorial ]
image: assets/images/nodejs-pm2.webp
description: "Monitoring Node.js Applications Using PM2"
comments: true
---

Node.js has become a popular choice for developing server-side applications due to its efficiency and scalability. However, as with any application, monitoring and managing Node.js processes are crucial for ensuring optimal performance and reliability. PM2 (Process Manager 2) is a versatile tool that simplifies the task of managing and monitoring Node.js applications.

### What is PM2?
PM2 is a production-ready process manager for Node.js applications. It simplifies the management of Node.js processes by providing features like process clustering, automatic restarts, and detailed monitoring. With PM2, you can ensure that your Node.js applications are always up and running, and you can easily scale them as needed.

### Installing PM2
To get started, you'll need to install PM2 globally on your server or development machine. You can do this using npm (Node Package Manager) with the following command
```
npm install pm2 -g
```

### Running a Node.js Application with PM2 in Fork mode
Once PM2 is installed, you can use it to start and manage your Node.js applications. Here's how to run a Node.js app using PM2
```
pm2 start app.js

[PM2] Spawning PM2 daemon with pm2_home=/Users/pratyush/.pm2
[PM2] PM2 Successfully daemonized
[PM2] Starting /Users/pratyush/git/nodejs-crud-mongo/app.js in fork_mode (1 instance)
[PM2] Done.
┌─────┬────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name   │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├─────┼────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0   │ app    │ default     │ 1.0.0   │ fork    │ 51518    │ 0s     │ 0    │ online    │ 0%       │ 10.7mb   │ pratyush │ disabled │
└─────┴────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
```
> PM2 will automatically manage the process, ensuring it stays running and restarting it if it crashes.

### Stopping and deleting the the Node.js Application from pm2 list
```
pm2 stop app
[PM2] Applying action stopProcessId on app [app](ids: [ 0 ])
[PM2] [app](0) ✓
┌─────┬────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name   │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├─────┼────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0   │ app    │ default     │ 1.0.0   │ fork    │ 0        │ 0      │ 0    │ stopped   │ 0%       │ 0b       │ pratyush │ disabled │
└─────┴────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘

pm2 delete all
[PM2] Applying action deleteProcessId on app [all](ids: [ 0 ])
[PM2] [app](0) ✓
┌─────┬───────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name      │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
└─────┴───────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
```

### Running a 3 node Node.js Application with PM2 in Cluster mode
The cluster mode allows networked Node.js applications (http(s)/tcp/udp server) to be scaled across all CPUs available, without any code modifications. This greatly increases the performance and reliability of your applications, depending on the number of CPUs available.
```
pm2 start app.js -i 3
[PM2] Starting /Users/pratyush/git/nodejs-crud-mongo/app.js in cluster_mode (3 instances)
[PM2] Done.
┌─────┬────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name   │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├─────┼────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0   │ app    │ default     │ 1.0.0   │ cluster │ 52010    │ 0s     │ 0    │ online    │ 12%      │ 30.9mb   │ pratyush │ disabled │
│ 1   │ app    │ default     │ 1.0.0   │ cluster │ 52011    │ 0s     │ 0    │ online    │ 11%      │ 29.3mb   │ pratyush │ disabled │
│ 2   │ app    │ default     │ 1.0.0   │ cluster │ 52016    │ 0s     │ 0    │ online    │ 6%       │ 19.7mb   │ pratyush │ disabled │
└─────┴────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
```

### Scaling by adding 1 more node
PM2 can automatically restart processes that crash or encounter issues. Additionally, it supports process scaling, allowing you to run multiple instances of your Node.js application to handle high traffic loads.
```
pm2 scale app +1
[PM2] Scaling up application
┌─────┬────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name   │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├─────┼────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0   │ app    │ default     │ 1.0.0   │ cluster │ 52295    │ 3m     │ 0    │ online    │ 0%       │ 44.3mb   │ pratyush │ disabled │
│ 1   │ app    │ default     │ 1.0.0   │ cluster │ 52296    │ 3m     │ 0    │ online    │ 0%       │ 44.8mb   │ pratyush │ disabled │
│ 2   │ app    │ default     │ 1.0.0   │ cluster │ 52299    │ 3m     │ 0    │ online    │ 0%       │ 43.6mb   │ pratyush │ disabled │
│ 3   │ app    │ default     │ 1.0.0   │ cluster │ 52339    │ 0s     │ 0    │ online    │ 0%       │ 19.6mb   │ pratyush │ disabled │
└─────┴────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
```

### Zero-downtime reloads
PM2 can Performs a graceful reload by reloading the workers one by one keeping the app online the whole time.
> Reload is perfect for cluster mode, rolling restarts, and production deployments.

```
pm2 reload app
Use --update-env to update environment variables
[PM2] Applying action reloadProcessId on app [app](ids: [ 0, 1, 2 ])
[PM2] [app](0) ✓
[PM2] [app](1) ✓
[PM2] [app](2) ✓
```

### Stopping and deleting the the Node.js Application from pm2 list
```
pm2 stop all 
[PM2] Applying action stopProcessId on app [all](ids: [ 0, 1, 2 ])
[PM2] [app](0) ✓
[PM2] [app](1) ✓
[PM2] [app](2) ✓
┌─────┬────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name   │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├─────┼────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0   │ app    │ default     │ 1.0.0   │ cluster │ 0        │ 0      │ 0    │ stopped   │ 0%       │ 0b       │ pratyush │ disabled │
│ 1   │ app    │ default     │ 1.0.0   │ cluster │ 0        │ 0      │ 0    │ stopped   │ 0%       │ 0b       │ pratyush │ disabled │
│ 2   │ app    │ default     │ 1.0.0   │ cluster │ 0        │ 0      │ 0    │ stopped   │ 0%       │ 0b       │ pratyush │ disabled │
└─────┴────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘

pm2 delete all
[PM2] Applying action deleteProcessId on app [all](ids: [ 0, 1, 2 ])
[PM2] [app](0) ✓
[PM2] [app](1) ✓
[PM2] [app](2) ✓
┌─────┬───────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name      │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
└─────┴───────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘

```

### PM2 essential monitoring commands
1. **View Process List**: To see a list of all running Node.js processes managed by PM2, use the following command
```
pm2 list
```
![pm2 list](/assets/images/pm2-list.png)

2. **Monitor CPU and Memory**: PM2 also provides a real-time monitoring dashboard. You can access it by running
```
pm2 monit
```
![pm2 monit](/assets/images/pm2-monit.png)

3. **Show Project Meta Data**: To see various information of the project such as Revision Control, Code metrics etc by running the command
```
pm2 describe 0
```
![pm2 describe](/assets/images/pm2-describe.png)

4. **Show logs information**: Command to check access and error logs
```
pm2 logs
```
![pm2 logs](/assets/images/pm2-logs.jpeg)

### Use of `ecosystem.config.js` file
The ecosystem.config.js file is a configuration file used with PM2 to defines the configuration settings for one or more Node.js applications that you want to manage using PM2. It allows you to specify various options and parameters related to how PM2 should run and manage your Node.js processes. A basic structure of the file consists of `apps` and `deploy`

```
module.exports = {
  apps: [{}, {}],
  deploy: {}
}
```

Here's an example of a minimal `ecosystem.config.js` file
```
module.exports = {
  apps: [
    {
      name: 'my-app',
      script: 'app.js',
      instances: 1,
      exec_mode: 'fork',
      autorestart: true,
      env: {
        NODE_ENV: 'production',
        PORT: 3000,
      },
    },
  ],
};
```

### Seamlessly than acting on an app you can start/stop/restart/delete all apps contained in a configuration file
```
# Start all applications
pm2 start ecosystem.config.js

# Stop all
pm2 stop ecosystem.config.js

# Restart all
pm2 restart ecosystem.config.js

# Reload all
pm2 reload ecosystem.config.js

# Delete all
pm2 delete ecosystem.config.js
```

Monitoring Node.js applications is essential to maintain their stability and performance. PM2 simplifies this task by providing a robust set of features for process management, monitoring, log management, and scaling.