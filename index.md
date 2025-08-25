---
layout: splash
title: "Securing an Apache Server"
excerpt: "A step-by-step guide to securing Apache with Fail2Ban and UFW"
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header.jpg
  actions:
    - label: "Get Started"
      url: "/securing-an-apache-server/fail2ban/"
toc: true
toc_sticky: true
---

# About This Guide

This document outlines how to harden a Linux server after an intrusion, or to prevent intrusions before they occur. You will learn how to troubleshoot common issues related to server security using Apache, Fail2Ban, and UFW. Organized by topic, each section addresses specific problems encountered and their solutions.

# Audience

This guide is intended for system administrators, DevOps engineers, and anyone responsible for maintaining the security of Apache servers. It assumes a basic understanding of server administration, Apache configuration, and Linux command-line operations.

## Table of contents

* [Discovering Server Intrusions by Bad Actors](/securing-an-apache-server/discovering-intrusions/)
* [Automatically Detect Intrusions with Fail2Ban](/securing-an-apache-server/fail2ban/)
* [Use Uncomplicated Firewall (UFW) to Block Bad Actors](/securing-an-apache-server/ufw/)
