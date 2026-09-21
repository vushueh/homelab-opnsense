# Q027 current backup and rule inventory — 2026-09-20

Leonel downloaded config-OPNsense.internal-20260920165023.xml into local Downloads. File size 130,183 bytes; SHA-256 799586fd4e59d49294ff790385d2345b9c87eec25f7f245b53247074d0906bce. OPNsense encrypted backup marker detected. It is not plaintext XML; parse failure is not evidence of corruption. Decryption, password recoverability and restore are untested. The backup stays outside Git; only metadata is retained here.

Read-only SSH inspected explicit configured rule overrides and generated rule-file action counts. These prove files/configuration present, not every signature loaded successfully at runtime. ICMP candidates require full matching-condition and active-rule verification before choosing a deterministic test. No synthetic traffic or configuration change was performed. Do not interpret the candidate list as a ready-to-run signature recipe.

Next gates: identify running owned Kali/VLAN40 source and VLAN250 target, confirm hypervisor console recovery access, and verify a suitable enabled rule and the Wazuh collection route. Preserve existing WAN capture pending scope review. Retain the encrypted backup password privately; no password or raw backup was sent to an agent or committed.
