# Disclaimer

**Last updated: 2026-10-18**

In this Disclaimer, "this Site" or "the Software" refers to the content site and related services built, deployed, and operated using PNote, "we" refers to the operator of that site (i.e., the deployer/administrator of this project), and "you" refers to any third party who accesses, browses, registers (if applicable), or otherwise uses this Site.

> Note: PNote is **self-hosted** software. Any site built with this software is independently operated and controlled by its respective deployer. The original author of this project is not responsible for the content or operation of third-party deployments.

---

## 1. Software Provided "As Is" — No Warranties

The Software is provided on an "AS IS" and "AS AVAILABLE" basis, without any express or implied warranties, including but not limited to warranties of **merchantability, reliability, availability, security, non-infringement, and fitness for a particular purpose**.

We make no promises or guarantees that:

- The Software will operate continuously, uninterrupted, or error-free;
- Any feature (such as article publishing, project showcasing, Git sync, AI assistance, search, or backup) will always be available or produce accurate results;
- The Software will be fully compatible with your operating system, hardware, browser, or network environment;
- Any content generated, converted, synced, or displayed through the Software (including AI-generated content) is correct, lawful, or usable.

---

## 2. Data and Backup Risks Are Your Responsibility

- You are solely responsible for backing up and safeguarding all content, files, source code, and repositories you upload, publish, store, or commit through this Site.
- Backup features (if provided) are **manually triggered** and depend on your correctly configured environment and storage. We do not guarantee that any backup will succeed, be complete, or be restorable at any time.
- We accept no liability for any data loss, corruption, or disclosure caused by hardware failure, storage media damage, misconfiguration, operator error, version upgrades, deletion of containers/volumes (e.g., `docker compose down -v`), security incidents, or force majeure.
- You understand and agree that deleting containers or data volumes, misconfiguring environment variables, or forgetting your `SECRET_KEY` may result in **permanent and unrecoverable data loss**.

---

## 3. You Are Independently Responsible for Your Content

- You bear full legal responsibility for any text, images, code, links, or other content you publish, upload, display, sync, or transmit through this Site.
- You represent that your content does not violate applicable laws and regulations, does not infringe any third party's intellectual property, portrait, reputation, trade-secret, or other lawful rights, and contains no viruses, malware, or unlawful information.
- Any dispute, claim, administrative penalty, or litigation arising from your content is **your sole responsibility to resolve and bear**, and we accept no joint or vicarious liability.
- If you sync this Site's content to third-party platforms (e.g., GitHub, GitLab, Gitee, self-hosted repos), you must comply with those platforms' rules and the laws of the relevant jurisdiction.

---

## 4. Third-Party Services and Dependencies

The Software may integrate with or depend on the following third-party services and components, for whose availability, security, and compliance we offer no warranty and accept no liability:

- **AI assistance**: If you configure Anthropic / OpenAI or other LLM APIs, your content (including article text) may be sent to the corresponding provider. Processing of such data is governed by that provider's privacy policy, and the risk is yours; AI-generated content is for reference only and is **not guaranteed accurate, reliable, or usable** — verify it yourself.
- **Code hosting and sync**: Features such as Git clone/push and one-way sync to external repositories depend on the target services you configure; their availability, rate limits, and data egress are determined by those services.
- **Diagram rendering**: Diagrams such as PlantUML depend on rendering services you deploy yourself; the official image does not bundle such a service and will degrade gracefully.
- **Runtime environment**: The Software depends on third-party components such as Docker, PostgreSQL, and Redis; vulnerabilities or failures of those components are outside the scope of the Software's responsibility.

Using the above third-party services is deemed your acceptance of their terms of service and privacy policies.

---

## 5. Security Configuration Is Your Responsibility

- The Software provides various security mechanisms (two-factor authentication, API tokens, CSP, HSTS, upload validation, hotlink protection, audit logs, etc.), but **security depends on your correct configuration** (e.g., setting a strong random `SECRET_KEY`, `CORS_ORIGINS`, `PUBLIC_BASE_URL`, and enabling `TRUSTED_PROXY` with a correct `PROXY_PEER` where applicable).
- If you use weak passwords, omit required configuration, expose the service on an insecure network, or fail to upgrade promptly to fix known vulnerabilities, we accept no liability for resulting security incidents (e.g., unauthorized access, data breaches, account compromise).
- The Software is positioned as a single-administrator site and lacks multi-user collaboration and fine-grained permissions; assess whether it fits your use case accordingly.

---

## 6. Limitation of Liability

To the maximum extent permitted by applicable law, we shall not be liable for any indirect, incidental, special, consequential, or punitive damages (including but not limited to loss of profits, data loss, business interruption, or reputational harm) arising from the use of or inability to use the Software, whether based on contract, tort, or any other legal theory, **even if advised of the possibility of such damages**.

If the laws of your jurisdiction do not allow the exclusion or limitation of certain liabilities, this Disclaimer applies only to the extent permitted by such laws.

---

## 7. Links and External Content

This Site may contain links to third-party websites or resources. These are provided for convenience only; we are not responsible for, and do not endorse, the content, privacy practices, or availability of any third-party website. You access third-party websites at your own risk.

---

## 8. Service Changes and Interruptions

We reserve the right to modify, suspend, restrict, or terminate any part or all of the Software's features at any time without notice, and accept no liability thereby. Public instances such as demo sites may differ in content, version, and availability from the latest source code and may be taken offline at any time.

---

## 9. Changes to This Disclaimer

We may update this Disclaimer from time to time. Updated terms take effect upon posting on this Site; we recommend reviewing it periodically. Continued use of this Site after an update constitutes acceptance of the revised terms.

---

## 10. Governing Law and Disputes

The interpretation and application of this Disclaimer, and any dispute arising from use of this Site, shall be governed by the laws applicable at the location of the Site operator or where your use occurs. If any provision of this Disclaimer is found invalid or unenforceable, the remaining provisions shall remain in effect.

---

*This Disclaimer is a general template and does not constitute legal advice. Consult a qualified attorney for legal risks.*
