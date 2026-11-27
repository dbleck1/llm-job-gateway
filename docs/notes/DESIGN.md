1. Purpose
a server that acts as a single entry point between clients and a backend
Requirements
2. Functional

Non-Functional


3. API

4. Job states

5. Components
    API-recieves clients tenant_id, idempotency key, and docs, then validates saving as queued to postgres and sends 202 accepted to tenant queue
    tenant queues- job_id is sent to a redis list works as a cache to contain the queued items
    dispatcher- takes the tenant from the job_id the redis provided when popped. it checks the tokens left in the tenants bucket using deficit-read rotation, if the budget or cost of the call is within budget it then sends job_id to server bus as the message id. 
    server bus- sends msg in peek-lock mode
    workers- marks job as running and calls LLM through the provider client 
    provide client- holds the circuit breaker and fails sending over to a second provider
    postgres- holds jobs information and accessed through job_id
    dlq- happens if failure occurs n times, service bus moves the message to dlq where it is marked af failed
6. Data model
jobs| type
id | UUID
tenant_id | UUID
idempotency_key | UUID
status | Enum 
payload |
result |
error | 
attempts | int
created_at | datetime
updated_at | datetime

7. Failure cases
    - A worker crashes mid-job- the queue assumes job was never finished and makes the msssage vsisible again. the job is then restarted from scratch 
    - A duplicate submit - client retries a timed-out http request and sends the same payload twice. we wil stop this using the idempotency key the api will detect duplicate keys then skip creating a new queue msg, so returns just job id or result
    - The LLM returns 429 - worker hits rate limit it is not a permanent failure not sure how we will solve this
    - The LLM provider is down 500/502/503 error- a circuit breaker stops workers from sending requests. workers will probably go to second provider or they will be held in a retry queue
    - A message keeps failing - eventually it will reach DLQ if failed enough times marking job as failed in db. this then allows other jobs to go through
    - the database is unreachable 08001/08006 503 Service unavaliable says temporarily uanble to handle requests 
8. Out of scope for now: auth, streaming and a UI