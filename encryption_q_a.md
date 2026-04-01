# How and Why: The Encryption System of Terminal Notes

## A Document of Discovery, Not Design

---

## Preface

This document explains the encryption system as it exists. It does not describe a design process that began with theory. It describes a process that began with constraints and needs, and discovered theory afterward.

The encryption system was not built by reading papers. It was built by answering questions:

- "What do I need?"
- "What do I have?"
- "What can go wrong?"
- "What would be easy to use?"
- "What would be hard to break?"

The answers shaped the system. The theory came later.

---

## 1. The First Question: What Do I Need?

**Need:** My notes should be private. Only I can read them.

**First thought:** Encrypt with a password. Everyone does this.

**Second thought:** Passwords are weak. People reuse them. People forget them. What if the password is "password"? What if I forget it?

**Third thought:** What if the password isn't the key? What if the password just unlocks the key?

This was the first fork. Most systems stop at "password = key". This system went the other way.

---

## 2. The Second Question: What Do I Have?

**I have:** A laptop. A folder on disk. A user who will type something daily.

**I also have:** A recovery phrase from cryptocurrency wallets. 12 words. Unbreakable.

**Constraint:** I cannot type 12 words every day. Too long. Too tedious.

**Insight:** The recovery phrase should be the real key. The password should be the daily convenience.

This separation—authentication vs. encryption—became the foundation.

---

## 3. The Third Question: What Is the Encryption Key?

If the password is not the key, what is?

**Option 1:** Use the phrase directly as the key.
- Problem: Phrase is 12 words. AES-256 expects 32 bytes. Need to convert.

**Option 2:** Hash the phrase into 32 bytes.
- Use SHA256. Standard. One-way. Output is exactly 32 bytes.

**But:** SHA256 is fast. Fast means an attacker can try many phrases per second. Is that a problem?

**Depends:** How many phrases can they try? 2^132 possibilities (12-word phrase). At 1 billion guesses per second, 2^132 / 1e9 = 10^30 years. The speed of SHA256 doesn't matter. The key space is too large.

**No iteration needed.** Entropy is the protection, not slowness.

---

## 4. The Fourth Question: What About the Folder Name?

The folder name is public. Should it be part of the key?

**If not:** Same phrase on two different notebooks = same key. Not ideal.

**If yes:** Same phrase on two different notebooks = different keys. Good.

**Result:** `key = SHA256(phrase + folder_name)`

The folder name becomes a salt. It's public. It doesn't need to be secret. It only needs to ensure uniqueness.

---

## 5. The Fifth Question: What Is the Password For?

If the notebook is encrypted with the phrase key, what does the password do?

**Answer:** It unlocks access to the phrase key.

**How:** Store the password key (Kp) encrypted with the phrase key (Ks). When user enters password, derive Kp, decrypt the stored Kp to verify. If correct, use Ks from cache to decrypt notebook.

**But:** Ks must be cached somewhere. Where?

**Answer:** In secure session, encrypted with system fingerprint.

---

## 6. The Sixth Question: What Is a System Fingerprint?

Every computer has unique identifiers:
- Linux: `/etc/machine-id`
- macOS: `IOPlatformUUID`
- Windows: `MachineGUID`

Combine these with hostname, username, platform info. Hash with SHA256. Result: 32 bytes that identify this machine.

**Use:** Encrypt the cached keys with this fingerprint. Keys cannot be moved to another machine.

**Trade-off:** If hardware changes, keys become inaccessible. But the phrase still works. User can re-import with phrase.

---

## 7. The Seventh Question: What If I Forget the Password?

The notebook is encrypted with the phrase key. If I have the phrase, I can decrypt it. But I need the password key to verify daily use.

**Solution:** Store the password key (Kp) encrypted with the phrase key (Ks) in `.tn_recovery`. During recovery:
1. User enters phrase
2. Derive Ks
3. Decrypt `.tn_recovery` to get Kp
4. Now have both keys
5. Set new password, update `.tn_recovery` and `.tn_password`

**Recovery works on any machine.** No cloud. No email. No support ticket.

---

## 8. The Eighth Question: What If I Change the Password?

The notebook is encrypted with Ks. Password only affects Kp.

**To change password:**
1. Derive new Kp from new password
2. Update `.tn_recovery` with new Kp (still encrypted with Ks)
3. Update `.tn_password` with new combined key
4. Update secure session with new Kp

**Notebook unchanged.** Password change is instant. No re-encryption.

---

## 9. The Ninth Question: What Is the Combined Key For?

The combined key (Kc = SHA256(Kp + Ks)) is used only for `.tn_password`.

Why? To verify that both Kp and Ks are correct. `.tn_password` is encrypted with Kc and contains Kc. Decrypting it with Kc proves Kc is correct, which proves Kp and Ks are correct.

**Self-referential verification.** Elegant. Simple. Secure.

---

## 10. The Tenth Question: What About File Size?

Encryption adds overhead:
- 12 bytes nonce (random)
- 16 bytes GCM tag

Total: 28 bytes per file. No base64. No JSON wrapper. Just raw encrypted bytes.

**Why not base64?** Base64 inflates size by 33%. No need. The files are binary. Git handles binary. GitHub handles binary. The terminal handles binary.

**Result:** 1KB note becomes ~1,028 bytes encrypted. Minimal overhead.

---

## 11. The Eleventh Question: What About Git?

Git works with binary files. Encrypted files are binary. Push to GitHub. Pull anywhere.

**Security:** Even if the repo is public, the files are encrypted. The only way to read them is with the phrase.

**Free sync:** GitHub becomes a free, encrypted sync service. No subscription. No limits. No vendor lock-in.

---

## 12. The Twelfth Question: What About the User Who Writes a Poem?

The phrase system accepts:
- 6 words (quick recovery)
- 8 words (balanced)
- 12 words (Bitcoin standard)
- 24 words (paranoid)
- Custom text (any length, any content)

A user can write: "The old oak tree behind my grandmother's house where I played as a child in the summer of 1998 with my dog Max and my sister Anna"

This is:
- Unique
- Memorable
- High entropy (no one else knows this story)
- Unbreakable

**The system doesn't force security. It enables it.**

---

## 13. The Thirteenth Question: What About the User Who Chooses a Weak Phrase?

The system allows 6-word phrases. 66 bits of entropy. At 1 million guesses per second, 2^66 / 1e6 = 2,000 years.

**Even the weakest option is strong enough for personal notes.** The user chooses their security level. The system does not force them to choose more than they need.

---

## 14. The Fourteenth Question: What About the User Who Loses the Phrase?

The phrase is the ultimate key. Lose it, lose the notebook on new machines. On machines where the notebook is already unlocked, it continues to work. But import on new machines requires the phrase.

**This is by design.** The user is responsible for the phrase. The system does not store it. The system cannot recover it.

**Trade-off:** Sovereignty vs. convenience. The user chooses.

---

## 15. The Fifteenth Question: What About the User Who Has Nothing?

This system was built by someone who had nothing. A laptop. A wooden chair. 120 days.

**The constraints shaped the system:**
- No cloud → use Git
- No money → use GitHub for free sync
- No test environment → make it portable, run from USB
- No dependencies → bundle everything
- No team → AI as collaborator
- No time → focus on what matters

**The system is a reflection of its constraints.**

---

## 16. The Sixteenth Question: Why Does This Work?

**Because entropy beats iteration.**
- PBKDF2 makes each guess slower
- A strong phrase makes the number of guesses astronomically larger

2^132 is 2^88 times larger than 2^44. That factor is 3 × 10^26. No amount of iteration can overcome a factor of 10^26.

**Because separation beats complexity.**
- Password is for daily use
- Phrase is for ultimate security
- Hardware binding prevents key theft
- Instant password changes because password isn't the key

**Because constraints clarify.**
- No cloud → Git is the sync layer
- No money → GitHub is free
- No dependencies → Python standard library is enough
- No time → focus on architecture, not features

---

## 17. The Logic Chain

The system follows a chain of reasoning:

1. I need privacy → encrypt
2. Passwords are weak → add a phrase
3. Phrase is too long for daily use → make password daily, phrase for recovery
4. Keys should stay on machine → bind to hardware fingerprint
5. Password changes should be instant → password is not the encryption key
6. Sync should be free → use Git, encrypted blobs
7. Files should be small → raw AES-GCM output, no base64
8. Recovery should work anywhere → phrase only, no cloud
9. User should own the key → phrase never stored
10. System should be portable → copy folder, phrase import

**Each step follows from the previous. No step is arbitrary.**

---

## 18. The Unintended Discovery

After building, I discovered that:
- The 28-byte overhead matches GCM's minimum
- The 2^256 key space matches Bitcoin
- The phrase system matches BIP-39
- The hardware binding matches PUF research
- The separation of authentication from encryption is a known pattern in security literature

**I didn't read these papers. I just built what made sense. The theory was already there, waiting to be discovered.**

---

## 19. The Humble Truth

This system is not novel in its primitives:
- AES-256-GCM is standard
- SHA256 is standard
- BIP-39 phrases are standard
- Hardware binding is known

**What is novel is the combination:**
- Password daily, phrase ultimate
- Hardware binding with system fingerprint
- Instant password changes
- No phrase storage
- Git as encrypted sync
- Portable folder import

The combination emerged from constraints, not from reading. It was discovered, not designed.

---

## 20. Conclusion

The encryption system exists because:
- I needed privacy
- I had a laptop and time
- I asked questions
- I followed logic
- I discovered what worked

The code is open. The documentation is complete. The user owns their data.

**This is not a claim of invention. It is a record of discovery.**

---

**sys_ronin**  
April 2026
