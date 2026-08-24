# Setup — apni notes website live karna

Yeh folder ready hai. Sirf push karna baaki hai.

> ⚠️ **Pehle yeh padho:** yeh setup **personal laptop pe karo, office laptop pe nahi.**
> Personal GitHub credentials office machine pe rakhna theek nahi hai, aur agar kabhi
> laptop wapas karna pada toh sab wahin reh jayega.
>
> **Aaj hi:** poora `qa-notes-site` folder pendrive ya personal Google Drive pe copy kar lo.
> Abhi yeh 6.2 MB **sirf office laptop pe hai** — koi backup nahi.

---

## Isme kya hai

```
qa-notes-site/
├── docs/                 <- 11 technical docs — YE PUBLISH HONGE
│   ├── index.md          <- homepage
│   ├── python.md
│   ├── dsa-patterns.md
│   └── ...
├── private/              <- 2 job-search docs — YE PUBLISH NAHI HONGE
│   ├── interview-strategy.md
│   └── market-research.md
├── mkdocs.yml            <- site config (theme, navigation, search)
├── requirements.txt
├── .gitignore            <- private/ ko ignore karta hai
└── .github/workflows/
    └── deploy.yml        <- push karo, site apne aap update
```

### `private/` alag kyu hai — yeh important hai

Woh do docs **public nahi hone chahiye**. Unme yeh sab likha hai:

- Company chhodne ki planning aur "1 saal tenure kaise justify karein" wala answer
- Salary bands aur negotiation
- Kaun si companies target karni hain

Agar yeh public site pe chala gaya, toh **Google pe search karne se koi bhi padh sakta hai** —
tumhara current manager bhi. `.gitignore` mein `private/` daala hua hai, toh galti se
push bhi nahi hoga.

Woh docs tumhare paas rahenge, bas website pe nahi jayenge.

---

## Step 1 — repo banao (personal GitHub account pe)

GitHub pe jao → **New repository**

- Name: `qa-notes`
- **Public** rakho (Google pe tabhi aayega; GitHub Pages private repo pe free nahi hai)
- README/`.gitignore` **mat** add karo — yahan already hai

---

## Step 2 — apna username daalo

`mkdocs.yml` kholo, `ritik` ko apne actual GitHub username se replace karo (3 jagah):

```yaml
site_url: https://TUMHARA-USERNAME.github.io/qa-notes/
repo_url: https://github.com/TUMHARA-USERNAME/qa-notes
...
    - icon: fontawesome/brands/github
      link: https://github.com/TUMHARA-USERNAME
```

LinkedIn wala link bhi update kar lena.

---

## Step 3 — push karo

```bash
cd qa-notes-site

git init
git add .
git commit -m "QA engineering notes"
git branch -M main
git remote add origin https://github.com/TUMHARA-USERNAME/qa-notes.git
git push -u origin main
```

Push karne se pehle confirm kar lo ki `private/` nahi ja raha:

```bash
git status --short | grep private || echo "OK — private/ ignore ho raha hai"
```

---

## Step 4 — GitHub Pages on karo

Repo → **Settings** → **Pages** → **Source** = `Deploy from a branch`
→ Branch = **`gh-pages`**, folder = `/ (root)` → **Save**

> `gh-pages` branch pehli deploy ke baad khud ban jaayegi. Agar abhi dikhe nahi,
> **Actions** tab pe jaake workflow complete hone do (~2 minute), phir wapas aao.

---

## Step 5 — live

`https://TUMHARA-USERNAME.github.io/qa-notes/`

Pehli baar 2–3 minute lagenge. Uske baad **har `git push` pe site apne aap update** ho jayegi.

---

## Roz ka use

Naya kuch padha? Ek markdown file likho aur push kar do:

```bash
# docs/ mein nayi file banao, phir:
git add . && git commit -m "notes on X" && git push
```

Nav mein dikhane ke liye `mkdocs.yml` ke `nav:` section mein entry add kar dena.

---

## Local pe preview (optional, par useful)

Push karne se pehle dekhna ho ki kaisa lag raha hai:

```bash
pip install -r requirements.txt
mkdocs serve
```

Browser mein `http://127.0.0.1:8000` kholo. File save karte hi live reload hota hai.

---

## Jo tumhe milega

| | |
|---|---|
| **Search** | Poore site mein, GeeksforGeeks jaisa |
| **Mobile** | Phone pe theek se khulega, bina login |
| **Google** | Index hoga — search se milega |
| **Dark mode** | Toggle button, viewer ke hisaab se |
| **Code copy** | Har code block pe copy button |
| **Tumhara** | Claude pe depend nahi, tumhara repo, tumhara URL |

Aur ek fayda jo abhi nahi dikh raha — **commit history**. Har push ek green square hai,
aur ek interview mein *"maine apni learning ka documentation site banaya hai"* ek real
cheez hai dikhane ke liye.

---

## Baad mein: apna domain

Domain khareedo (₹800/saal ke aas-paas), repo mein `docs/CNAME` file banao jisme sirf
domain likha ho, aur domain ke DNS mein GitHub Pages ke IP point kar do.
Phir `notes.tumhara-naam.com` jaisa URL ho jayega.
