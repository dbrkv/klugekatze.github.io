---
title: "Passing CKAD exam in 2021"
date: "2021-11-24"
tags:
    - ckad
    - exam
    - kubernetes
---

Yesterday I took Certified Kubernetes Application Developer (CKAD) exam. Probably, passing the exam in 2021 is not more different than in any other year I guess.

You have to be skilled in kubectl, and holy VIM. I'm practising with k8s daily, but had a very small experience hacking with kubectl imperative commands, editing YAML in Vi editor. Overall exam experience is remembered me CounterStrike gaming experience: strict timings, muscle memory on tips of you fingers to execute actions automatically. Just like on Dust make headshot of an enemy from AWP in jump between the gate leaves. Your skills in kubectl, hacking yaml's terminal with vim, ability to scan Kubernetes documentation blazing fast and precise are a huge portion of success.

## Reduce your weak points

If you don't know how to run a container as a user or mount secret as a volume, others - learn and practice.

You may not use, and, perhaps, never use, this in practice, but you have to know it, as its basics. With a 100% guarantee, such questions will be on your task list.

## Practice with Killer.sh

After the purchase of the exam, you have 2 sessions on killer.sh. Use your first session after you think you are ready for an exam. Activate the session in the early morning as the session is valid for 36 hours so you can run 2-3 times this day and the next day. I recommend activating second session after you improve your skills after the first attempt and 2-3 days before the scheduled exam. If you've done a good job on the mistakes, you'll see a big difference.

## Learn that VIM finally

I never use vim, and prefer to nano, if it's terminal. However, I should admit, it's more convenient with it's shortcuts and modes than nano after your panic "How to exit from here?" is over. Watch [Vim Crash Course video](https://youtu.be/knyJt8d6C_8) and bookmark [Vim cheatsheet](https://devhints.io/vim)
For editing YAML, memorize vim config keys and put them into `~/.vimrc`:

```vimrc
# ~/.vimrc
set tabstop=2
set expandtab
set shiftwidth=2
syntax on
```

## Scheduling exam and exam day

Schedule an exam early morning, when your mind is still fresh. Before exam "prewarm" yourself by executing practising tasks on your local minikube. "Prewarm" is not important, but it will set you up for a work rhythm for an exam. 

Solve tasks out of their order. I scrolled all questions first to classify them by difficulty and score, marked all tasks I skipped and started from somewhere in the middle of the list. 

Read task descriptions carefully, copy-paste all variables, namespace names and sections of YAML you found in the documentation.

## Recommended resources

Open Source Curriculum for CNCF Certification Courses

https://kubernetes.io/ - obviously contains absolutely everything you need to prepare for the exam

Introduction to Kubernetes (LFS158x)

Learn DevOps: The Complete Kubernetes Course

Kubernetes Certified Application Developer (CKAD) with Tests

NetworkPolicy recipes https://github.com/ahmetb/kubernetes-network-policy-recipes

## Personal summary

It's my first certification exam. I purchased it in late 2020 but didn't allocate time during a year to take it and postponed it till the latest days before expiration. Overall, The exam is a good way to learn topics deeper and streamline knowledge.


