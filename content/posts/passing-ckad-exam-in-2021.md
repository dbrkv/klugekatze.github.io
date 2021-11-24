---
title: "Passing CKAD exam in 2021"
date: "2021-11-24"
tags:
    - ckad
    - exam
    - kubernetes
---

![Certified Kubernetes Application Developer](/assets/passing-ckad-exam-in-2021-banner.png)

Yesterday I took Certified Kubernetes Application Developer (CKAD) exam. Passing the exam in 2021 is not more different than in any other year I guess.

The exam is not difficult, especially if you have practical experience with Kubernetes. In this post, I share what and how I did.

## Reduce your weak points

If you don't know how to run a container as a user or mount secret as a volume, others - learn and practice.

You may not use, and, perhaps, never use, this in practice, but you have to know it, as its basics. With a 100% guarantee, such questions will be on your task list. Learn how to implement deployment patterns using kubernetes primitives: using sidecars, route traffic to pods.

## Recommended resources

* [Open Source Curriculum for CNCF Certification Courses](https://github.com/cncf/curriculum)
* [Kubernetes documentation](https://kubernetes.io/) - obviously contains absolutely everything you need to prepare for the exam
* [Introduction to Kubernetes (LFS158x)](https://training.linuxfoundation.org/training/introduction-to-kubernetes/) - base course on edx
* [Learn DevOps: The Complete Kubernetes Course](https://www.udemy.com/course/learn-devops-the-complete-kubernetes-course/) - good live terminal demos
* [Kubernetes Certified Application Developer (CKAD) with Tests](https://www.udemy.com/course/certified-kubernetes-application-developer/) - subject explained with slides, practical labs
* [NetworkPolicy recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes) - recipes with visual explainations
* [CKAD Exercises](https://github.com/dgkanatsios/CKAD-exercises)

## Practice with killer.sh

After the purchase of the exam, you have 2 sessions on killer.sh. Use your first session after you think you are ready for an exam. Activate the session in the early morning as the session is valid for 36 hours so you can run 2-3 times this day and 1-2 the next day. I recommend activating the second session after you improve your skills after the first attempt and 2-3 days before the scheduled exam. If you've done a good job on the mistakes, you'll see a big difference.

## Learn that VIM finally

I always used nano instead of Vim for in terminal shell work. It's always been a panic mode "How to exit from this editor?". However, I should admit, it's more convenient with its key shortcuts and modes than nano. Now I'm started to use Vim. Watch [Vim Crash Course video](https://youtu.be/knyJt8d6C_8) and bookmark [Vim cheatsheet](https://devhints.io/vim)

For editing YAML, memorize vim config keys and put them into `~/.vimrc`:

```shell
# ~/.vimrc
set tabstop=2
set expandtab
set shiftwidth=2
syntax on
```

## Prepare bookmarks

I used 2 sets of bookmarks: my own and from [Alta3](https://alta3.com/blog/passing-the-ckad)(download them at the bottom). I had a set of approximately 20 of my own bookmarks pointing to some specific parts, like: security context, mount all env variables from a secret, mount single env variable from the secret key, etc.

## Scheduling exam and exam day

**Schedule an exam early morning**, when your mind is still fresh. Before exam "prewarm" yourself by executing practising tasks on your local minikube. "Prewarm" is not important, but it will set you up for a work rhythm for an exam.

**For macOS:** you have to grant permissions from system preferences to record screen for Chrome. Without this, Chrome chrome is not able to share screen **and** not reporting root cause why.

**Solve tasks out of provided order!** I scrolled all questions first to classify them by difficulty and score, marked all tasks I skipped and started from somewhere in the middle of the list.

**Read task descriptions carefully**, copy-paste all variables, namespace names and sections of YAML you found in the documentation. It's **faster** and **safer** to copy-paste, rather than return to correct typo "srviceAccountName" of "serviceAccountName"
You are allowed to open one tab with documentation, I extracted that tab into a new window and put windows one above another with the shift. Make Chrome windows not full-screen by width, a bit narrower, which allows you more quickly navigate between windows using the cursor by clicking the wide area at the right or left. You could also use 2nd monitor.

![Chrome window layouts with documentation and exam window](/assets/passing-ckad-exam-in-2021-window-layout-terminal.png)

![Chrome window layouts with documentation and exam window](/assets/passing-ckad-exam-in-2021-window-layout-docs.png)

## Personal summary

It's my first certification exam. I purchased it in late 2020 but didn't allocate time during a year to attempt it and postponed it till the latest days before expiration.

You have to be skilled in kubectl, and holy VIM. I'm practising with k8s daily, but had a very small experience hacking with kubectl imperative commands, editing YAML in Vi editor. Exam experience is reminded me Counter-Strike gaming experience: strict timings, muscle memory on tips of your fingers to execute actions automatically. Just like on Dust 2 make a headshot of an enemy from AWP in a jump between the gate leaves. Your skills in kubectl, hacking yaml's terminal with vim, ability to scan Kubernetes documentation blazing fast and precise are a huge portion of success, but knowledge is the primary key to passing en exam.

Overall, The exam is a good way to learn topics deeper and arrange knowledge.
