# Easy guide — install DarkMoon in the cloud

> **Easy-read guide.** Short sentences, simple words. Hard words are explained.

DarkMoon **is not a box you buy**. It is software.
You put it on a computer that you rent in the cloud.
And **one single command does everything**: it creates the computer, opens the door, and installs DarkMoon.

---

## Before you start, you need 3 things

1. Your DarkMoon **license key**. *(You find it in your client portal.)*
2. Your **AI API key**. *(This is a code given by your AI provider.)*
3. A **cloud account**: AWS, Google Cloud, Azure **or** OVH.
   *The cloud = computers that you rent over the internet.*

---

## The automatic way (one single command)

### Step 1 — Open your cloud's CloudShell
- In your cloud's console, click the **CloudShell** button.
  *CloudShell = a small black window, already signed in to your account. Nothing to install.*

### Step 2 — Paste one command
- In your **client portal**, open the **"Deploy to cloud"** card.
- Take the **① Fully automated** option. Click **Copy**.
- **Paste** the command into CloudShell.
- Replace `<YOUR_LLM_API_KEY>` with **your AI key**.
- For another cloud: replace `aws` with `gcp`, `azure` or `ovh`.
- Press **Enter**.

### Step 3 — Wait
- The computer **creates itself**.
- The door (port 80) **opens by itself**.
- DarkMoon **installs itself**. This takes **about 3 minutes**.
- The dashboard **address appears** on screen.

### Step 4 — Open DarkMoon
- Copy the address shown into your browser.
- The DarkMoon dashboard opens. **It is ready.** 🎉

---

## What happens on its own

- ✅ Create the computer in the cloud.
- ✅ Open the door (port 80).
- ✅ Install Docker.
- ✅ Install DarkMoon.

You **do not** have to create the machine by hand.
You **do not** have to open the door by hand.
You **do not** have to connect over SSH.

---

## The only thing we cannot do for you

You must be **signed in to YOUR cloud account**.
CloudShell does this for you: you are already signed in.

We **never take** your cloud credentials.
**You** run the command, in **your** account. That is safer.

---

## If something does not work

- Connect to your machine.
- Type: **`darkmoon doctor`**
- Doctor checks everything and tells you what to do.

---

## Choosing the machine size

The command uses a default size ("standard").
You can pick another one with `--profile`:

| Need | Write | Cores / Memory |
|---|---|---|
| Small (trial) | `--profile minimum` | 2 / 8 GB |
| Normal | `--profile standard` | 4 / 16 GB |
| Powerful | `--profile performance` | 8 / 32 GB |
| Industrial | `--profile industrial` | 16 / 64 GB |

---

## Going further (for technical people)

- The full command and the 3 methods: see [one-command.md](one-command.md).
- Per cloud: [AWS](aws.md) · [GCP](gcp.md) · [Azure](azure.md) · [OVH](ovh.md).
- Security (put the keys in a vault): [security.md](security.md).
