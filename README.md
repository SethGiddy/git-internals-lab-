# Git Internals Lab

Ever wondered what actually happens when you commit? Or why Git is so powerful at recovering lost work? This lab walks you through Git's guts—the real data structures and files that make version control magic happen.

## The Big Picture

Here's the secret: Git doesn't think in terms of filenames or "who changed what." Instead, it stores everything by **content hash**. Every file, every folder structure, every commit—they all get a unique fingerprint. That `.git` folder in your project? That's the entire universe of your repository. It contains the database of blobs (actual file content), trees (folder structures), commits (snapshots in time), and refs (pointers to branches).

## Let's Get Started

```bash
mkdir git-internals-lab && cd git-internals-lab
git init
git status
```

You just created an empty `.git` folder. That's it—that's your repository!

## Peek Inside the `.git` Folder

Want to see what Git actually created?

```bash
ls -la .git                    # You'll see HEAD, config, objects, refs, hooks
cat .git/HEAD                  # Points to your current branch
cat .git/config                # Your repository's settings
ls .git/refs                   # Where branch pointers live
```

It's all just files and folders. Boring, right? But wait...

## The Hash Magic

Let's create a file and watch Git fingerprint it:

```bash
echo 'Hello Git' > hello.txt
git hash-object hello.txt      # Shows the SHA fingerprint (but doesn't save it)
git hash-object -w hello.txt   # Now Git stores it in .git/objects
find .git/objects -type f      # See it's actually there!
git cat-file -t <hash>         # What type is this thing? (It's a "blob")
git cat-file -p <hash>         # Show me the contents
```

**Why this matters:** Git uses content-addressable storage. The same content always produces the same hash. Change even one character, and the hash is completely different. This is why Git history is tamper-proof.

## Your First Real Commit

Now let's create an actual commit with history:

```bash
git config --global user.email 'you@example.com'
git config --global user.name 'Your Name'
git add hello.txt
git commit -m 'Initial commit'
git log --oneline
```

Behind the scenes, Git just created *three* new objects: a blob (the file), a tree (the folder snapshot), and a commit (the metadata).

## Digging Into Commits

Let's peek at what a commit really looks like:

```bash
git rev-parse HEAD             # Get the commit's hash
git cat-file -p HEAD           # Show the full commit object
git ls-tree HEAD               # See the directory structure
```

A commit is just metadata (who, when, message) + a pointer to a complete snapshot of your project at that moment. It doesn't store "changes"—it stores a full picture.

## The Snapshot Trick

Here's what makes Git reliable:

```bash
find .git/objects -type f | wc -l    # Count objects
echo 'Another line' >> hello.txt
git add hello.txt && git commit -m 'Updated file'
find .git/objects -type f | wc -l    # Three NEW objects created
```

Every commit is a completely independent snapshot. Old commits never change. This is why you can instantly jump to any point in history, and why corruption doesn't cascade.

## Branches: Cheap Pointers

Here's the mind-bender about branches:

```bash
cat .git/HEAD                    # Just a text file: "ref: refs/heads/main"
cat .git/refs/heads/main         # Just a text file: the commit SHA
```

That's it. Branches are literally just text files with commit hashes. When you create a branch, Git doesn't copy anything—it just creates a new pointer. When you commit, Git updates the branch file to point to the new commit.

## Share Your Work

Ready to send it to the world?

```bash
ssh-keygen -t ed25519 -C 'you@example.com'
cat ~/.ssh/id_ed25519.pub             # Copy this to GitHub
git remote add origin git@github.com:<YOUR_USERNAME>/git-internals-lab.git
git push -u origin main
git ls-remote origin                  # Verify it's there
```

**What just happened:** Git found all your objects (blobs, trees, commits) and replicated them to GitHub's servers. The same SHA hashes, the same structure, the same history. Your local repo and the remote repo are now identical.

## The Aha Moments

- **Blobs** are just file content—they don't know their own filename
- **Trees** are like zip files—they map filenames to blobs
- **Commits** are like photos—complete snapshots with metadata
- **Refs** are like sticky notes—they point to commits
- **Hashes rule everything**—same content = same hash, everywhere
- **Nothing ever gets deleted**—old commits stay forever (unless you explicitly prune)
- **History is tamper-proof**—touch one byte, and every future hash breaks

## Why This Matters

Understanding Git's internals means you can:
- Recover "lost" commits (they're not really lost—they're still in `.git/objects`)
- Debug weird merge conflicts
- Clean up history surgically
- Understand why Git is distributed by default

Start small, run these commands, peek around, and you'll develop a sixth sense for how Git actually works.

Happy hacking! 🚀
