# Event Data Integration

Full Crossref Event Data pipeline using Docker. For integration testing, demonstration, reference implementation. Doesn't include any Crossref Agents. Can be used to test at any stage of the pipeline:

 - Test Agents pushing data into Percolator, through to the Live Demo display

This repository pulls together the core parts of the Crossref Event Data system using Git submodules and Docker Compose. Images are build from dockerfiles in submodules, not taken from a docker repository.

## To run

 1. Create `.env` file with `S3_BUCKET_NAME`, `S3_REGION_NAME` and `S3_SECRET`.
 2. `docker-compose up`

The following services are exposed on localhost to the host machine:

 - Status Service: http://localhost:8003
 - Live Demo: http://localhost:8101
 - Event Bus: http://locahost:8002
 - Percolator (accept): http://localhost:8006

Note that first time round, lots of dependencies will be fetched so startup may take some time.

## To update

  git submodule update --recursive --remote
  docker-compose build

## Demos

The very handy `jq` utility is suggested.

### View Thing Action Service

http://localhost:8003/thing-action-service/index.html

curl --verbose --header "Content-Type: application/json" --data "{\"total\":1,\"id\":\"d24e5449-7835-44f4-b7e6-289da4900cd0\",\"message_action\":\"create\",\"subj_id\":\"https:\\/\\/es.wikipedia.org\\/wiki\\/Se%C3%B1alizaci%C3%B3n_paracrina\",\"subj\":{\"pid\":\"https:\\/\\/es.wikipedia.org\\/wiki\\/Se%C3%B1alizaci%C3%B3n_paracrina\",\"title\":\"Se\\u00f1alizaci\\u00f3n paracrina\",\"issued\":\"2016-09-25T23:58:58.000Z\",\"URL\":\"https:\\/\\/es.wikipedia.org\\/wiki\\/Se%C3%B1alizaci%C3%B3n_paracrina\",\"type\":\"entry-encyclopedia\"},\"source_id\":\"wikipedia\",\"relation_type_id\":\"references\",\"obj_id\":\"https:\\/\\/doi.org\\/10.1093\\/EMBOJ\\/20.15.4132\",\"occurred_at\":\"2016-09-25T23:58:58Z\"}"  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyIxIjoiMSIsInN1YiI6Indpa2lwZWRpYSJ9.w7zV2vtKNzrNDfgr9dfRpv6XYnspILRli_V5vd1J29Q" http://localhost:8101/events

### View Live Demo

http://localhost:8101/live.html

Send directly to the Live Demo for testing

    curl --verbose --header "Content-Type: application/json" --data "{\"total\":1,\"id\":\"d24e5449-7835-44f4-b7e6-289da4900cd0\",\"message_action\":\"create\",\"subj_id\":\"https:\\/\\/es.wikipedia.org\\/wiki\\/Se%C3%B1alizaci%C3%B3n_paracrina\",\"subj\":{\"pid\":\"https:\\/\\/es.wikipedia.org\\/wiki\\/Se%C3%B1alizaci%C3%B3n_paracrina\",\"title\":\"Se\\u00f1alizaci\\u00f3n paracrina\",\"issued\":\"2016-09-25T23:58:58.000Z\",\"URL\":\"https:\\/\\/es.wikipedia.org\\/wiki\\/Se%C3%B1alizaci%C3%B3n_paracrina\",\"type\":\"entry-encyclopedia\"},\"source_id\":\"wikipedia\",\"relation_type_id\":\"references\",\"obj_id\":\"https:\\/\\/doi.org\\/10.1093\\/EMBOJ\\/20.15.4132\",\"occurred_at\":\"2016-09-25T23:58:58Z\"}"  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyIxIjoiMSIsInN1YiI6Indpa2lwZWRpYSJ9.w7zV2vtKNzrNDfgr9dfRpv6XYnspILRli_V5vd1J29Q" http://localhost:8101/events

### View Pipeline

NB: With the Agent refactor, this needs a change to work with the new event types.

http://localhost:8003/thing-action-service/pipeline.html

### Send an Event to the Bus

Note the `$RANDOM`, expanded by Bash, which ensures a different event ID is sent each time.

    curl --verbose --header "Content-Type: application/json" --data "{\"total\":1,\"id\":\"d24e5449-7835-44f4-b7e6-289da4900cd0$RANDOM\",\"message_action\":\"create\",\"subj_id\":\"https:\\/\\/es.wikipedia.org\\/wiki\\/Se%C3%B1alizaci%C3%B3n_paracrina\",\"subj\":{\"pid\":\"https:\\/\\/es.wikipedia.org\\/wiki\\/Se%C3%B1alizaci%C3%B3n_paracrina\",\"title\":\"Se\\u00f1alizaci\\u00f3n paracrina\",\"issued\":\"2016-09-25T23:58:58.000Z\",\"URL\":\"https:\\/\\/es.wikipedia.org\\/wiki\\/Se%C3%B1alizaci%C3%B3n_paracrina\",\"type\":\"entry-encyclopedia\"},\"source_id\":\"wikipedia\",\"relation_type_id\":\"references\",\"obj_id\":\"https:\\/\\/doi.org\\/10.1093\\/EMBOJ\\/20.15.4132\",\"occurred_at\":\"2016-09-25T23:58:58Z\"}"  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyIxIjoiMSIsInN1YiI6Indpa2lwZWRpYSJ9.w7zV2vtKNzrNDfgr9dfRpv6XYnspILRli_V5vd1J29Q" http://localhost:8002/events

### Send data to the Percolator

Note that depending on config, the Percolator may mark Actions as duplicates, so tweak their IDs before sending.

    curl http://localhost:8006/input --verbose --data @percolator-demo.json --header "Content-type: application/json" -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyIxIjoiMSIsInN1YiI6Indpa2lwZWRpYSJ9.w7zV2vtKNzrNDfgr9dfRpv6XYnspILRli_V5vd1J29Q" | jq .

