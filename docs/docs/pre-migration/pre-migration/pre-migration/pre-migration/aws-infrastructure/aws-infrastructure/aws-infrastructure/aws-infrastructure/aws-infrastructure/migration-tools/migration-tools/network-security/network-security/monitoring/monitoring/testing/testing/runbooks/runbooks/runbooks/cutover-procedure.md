# Migration Cutover Procedure

**Document Type:** Migration Runbook
**Author:** George Amankwaa Sarpong
**Last Updated:** June 2026

---

## Purpose
Step-by-step procedure for executing the final production cutover from on-premises infrastructure to AWS ensuring near-zero downtime and successful migration completion.

---

## Pre-Cutover Requirements
- All validation tests passed on AWS test instances
- Database replication lag below 5 seconds consistently
- Change request approved minimum 48 hours in advance
- All stakeholders notified of cutover window
- Rollback procedure confirmed and tested
- On-call team assembled and briefed
- Communication bridge established

---

## Cutover Timeline

| Time | Activity | Owner |
|---|---|---|
| T-24 hours | Reduce DNS TTL to 60 seconds | Cloud Engineer |
| T-4 hours | Final validation on AWS instances | Cloud Engineer |
| T-2 hours | Stakeholder go/no-go meeting | Project Manager |
| T-1 hour | Freeze changes on on-premises | Operations Manager |
| T-0 | Begin cutover | Cloud Engineer |
| T+15 min | DNS cutover complete | Cloud Engineer |
| T+30 min | Validation complete | QA Team |
| T+60 min | Cutover confirmed successful | Project Manager |

---

## Cutover Steps

### Phase 1 — Pre-Cutover (T-1 hour)
1. Confirm database replication lag below 5 seconds
2. Take final snapshot of on-premises database
3. Freeze all changes on on-premises systems
4. Notify all users of maintenance window
5. Confirm AWS instances are running and healthy
6. Verify CloudWatch monitoring is active

### Phase 2 — MGN Cutover
1. Open AWS MGN console
2. Select all source servers
3. Click Initiate Cutover
4. Wait for cutover instances to launch
5. Verify cutover instances running and healthy
6. Run application smoke tests

### Phase 3 — DNS Cutover
1. Open Route 53 console
2. Update A record from on-premises IP to AWS ALB DNS
3. Save changes
4. Monitor DNS propagation using dig command:

5. 5. Confirm DNS resolving to AWS ALB IP
6. Test application access from external network

### Phase 4 — Validation
1. Test all critical application functions
2. Verify database connectivity and data integrity
3. Check all integrations and APIs
4. Monitor CloudWatch dashboards for 30 minutes
5. Confirm no errors in application logs
6. Verify performance meets baseline requirements

### Phase 5 — Completion
1. Declare cutover successful
2. Notify all stakeholders
3. Update change request to completed
4. Begin monitoring phase — 24 hours enhanced monitoring
5. Schedule on-premises decommission review

---

## Go/No-Go Criteria

### Go Criteria (All must be met)
- [ ] Application accessible on AWS
- [ ] Database lag below 5 seconds
- [ ] All smoke tests passing
- [ ] CloudWatch metrics normal
- [ ] No critical errors in logs

### No-Go Criteria (Any triggers rollback)
- [ ] Application unavailable
- [ ] Database replication failed
- [ ] Performance degradation above 50%
- [ ] Data integrity issues found
- [ ] Critical application errors
