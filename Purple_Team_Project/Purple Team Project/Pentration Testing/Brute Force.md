# SSH Password Guessing with Hydra

## 1. Objective

Test the security of a lab SSH account against password-guessing attempts using Hydra in an authorized lab environment.

---

## 2. Environment

|Component|Details|
|---|---|
|Attacker|Kali Linux|
|Target|Windows SSH Server|
|Target IP|`192.168.255.148`|
|Service|SSH|
|Port|TCP/22|
|Tool|Hydra|
|Authentication Method|SSH Password Authentication|

---

## 3. Procedure

1. Identify the SSH service running on the target machine.
    
2. Use Hydra to perform a controlled password-guessing test against the lab SSH account using a wordlist.
    
3. Review the Hydra terminal output.
    
4. Record any valid credential pair reported by the tool.
    
5. Examine incomplete worker connections and determine whether the test completed successfully.
    
6. Review the relevant authentication logs and Wazuh alerts, where available.
    

---

## 4. Results

Hydra reported **one valid credential pair** for the account `global`.

However, some worker connections did not complete, meaning that the remaining password attempts were inconclusive.

### Result Summary

| Field                           | Result                 |
| ------------------------------- | ---------------------- |
| Target                          | `192.168.255.148`      |
| Service                         | SSH (TCP/22)           |
| Account                         | `global`               |
| Valid Credential Pairs Reported | 1                      |
| Test Completion                 | Partial / Inconclusive |
| Evidence Source                 | Hydra terminal output  |
| Screenshot                      | Not saved              |


---

## 5. Security Impact

Weak or easily guessed passwords can expose SSH accounts to unauthorized access when password authentication is enabled and the SSH service is reachable.

If an attacker obtains valid credentials, they may be able to authenticate to the target and access resources according to the account's permissions.

Potential risks include:

- Unauthorized access to the SSH account.
    
- Exposure of files and services accessible to the account.
    
- Further system compromise if the account has excessive privileges.
    
- Increased risk from repeated password-guessing attempts against exposed SSH services.
    

---

## 6. Recommendations

### Authentication Security

- Enforce strong, unique passwords for all SSH accounts.
    
- Prefer SSH key-based authentication where appropriate.
    
- Disable password authentication when operationally feasible and after verifying that key-based access works correctly.
    
- Apply least-privilege principles to SSH accounts.
    

### Network Security

- Restrict SSH access to trusted hosts or approved management networks.
    
- Avoid exposing SSH unnecessarily to untrusted networks.
    
- Apply firewall rules to limit access to TCP/22.
    

### Monitoring and Detection

- Monitor repeated failed SSH authentication attempts.
    
- Review Windows OpenSSH authentication logs.
    
- Investigate successful logins following repeated authentication failures.
    
- Review Wazuh alerts and authentication events for suspicious activity.
    
- Consider alerting on repeated failures against a single account or multiple accounts from one source.
    

---

## 7. Evidence

**Evidence Source:** Hydra terminal output.

**Screenshot:** Not saved.

The recorded terminal output reported one valid credential pair for the account `global`. No screenshot was retained for this test.

For future tests, save the terminal output or a screenshot containing the relevant results. Store any discovered credentials securely and redact passwords from reports, screenshots, and public repositories.

---

## 8. Limitations

- Some Hydra worker connections did not complete.
    
- The remaining password attempts were inconclusive.
    
- No screenshot was saved to independently document the terminal output.
    
- The Hydra result alone does not establish whether the successful authentication was recorded or detected by Wazuh.
    
- Defensive detection must be verified using the relevant authentication logs and security alerts.
    

---

## 9. Conclusion

The controlled SSH password-guessing test identified one valid credential pair for the lab account `global`, according to Hydra's terminal output.

Incomplete worker connections limited the scope of the test, so the remaining attempts were inconclusive.

The result highlights the importance of strong authentication, restricted SSH access, and centralized monitoring. Further investigation of authentication logs and Wazuh alerts is required to determine whether the activity was detected by the defensive monitoring stack.