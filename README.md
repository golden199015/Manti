
databases:
  - name: rudderstack
    ipAllowList: []
    plan: standard
    region: frankfurt

services:
  - type: web
    env: docker
    name: rudderstack-backend
    dockerfilePath: ./backend/dockerfile
    dockerContext: ./backend
    healthCheckPath: /
    plan: starter plus
    region: frankfurt
    envVars:
      - key: PORT
        value: 8080
      - key: JOBS_DB_HOST
        fromDatabase:
          name: rudderstack
          property: host
      - key: JOBS_DB_USER
        fromDatabase:
          name: rudderstack
          property: user
      - key: JOBS_DB_PORT
        fromDatabase:
          name: rudderstack
          property: port
      - key: JOBS_DB_DB_NAME
        fromDatabase:
          name: rudderstack
          property: database
      - key: JOBS_DB_PASSWORD
        fromDatabase:
          name: rudderstack
          property: password
      - key: CONFIG_BACKEND_URL
        value: https://api.rudderlabs.com
      - key: DEST_TRANSFORM_HOSTPORT
        fromService:
          name: rudderstack-transformer
          type: pserv
          property: hostport
      - key: WORKSPACE_TOKEN
        sync: false # have to manually enter in UI
      - key: STATSD_SERVER_URL
        fromService:
          name: rudderstack-statsd
          type: pserv
          property: hostport

  - type: pserv
    env: docker
    name: rudderstack-transformer
    dockerfilePath: ./transformer/dockerfile
    dockerContext: ./transformer
    plan: starter plus
    region: frankfurt
    envVars:
      - key: STATSD_SERVER_HOST
        fromService:
          name: rudderstack-statsd
          type: pserv
          property: host
      - key: STATSD_SERVER_PORT
        fromService:
          name: rudderstack-statsd
          type: pserv
          property: port

  - type: pserv
    env: docker
    name: rudderstack-statsd
    dockerfilePath: ./statsd/dockerfile
    dockerContext: ./statsd
    plan: starter
    region: frankfurt