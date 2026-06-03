+++
title       = "Building a physical security monitoring system — and what it taught me about SOC work"
date        = 2026-06-03
draft       = false
tags        = ["physical security", "unifi", "detection engineering", "home lab", "incident response"]
description = "How I designed and deployed an 11-camera 4K Unifi surveillance system with AI detection across a 40-room nonprofit facility — and why the lessons map directly to security operations."
+++

Physical security and cybersecurity are usually treated as separate disciplines. One involves cameras and locks, the other involves SIEMs and firewalls. But after designing and deploying a full surveillance system at a regional nonprofit, I came away with a clearer understanding of how tightly the two connect — and how much the operational challenges of physical security monitoring mirror the work of a SOC analyst.

## The deployment

The system covers LRDA's facility — a building with around 40 rooms across three entrances (front, rear, and side), with coverage both inside and outside.

**Hardware:**
- 6x Unifi G6 Bullet cameras (outdoor perimeter)
- 4x Unifi G6 Turret cameras (indoor coverage)
- 1x Unifi G6 360 Pro (wide-area coverage)
- 1x Unifi Viewport (dedicated display)
- 1x Unifi NVR with 20TB storage in RAID 5

Eleven cameras total, all 4K. The NVR hosts everything on-site — no cloud dependency, no third-party data handling. The RAID 5 configuration dedicates one drive to redundancy, giving the system fault tolerance without sacrificing too much usable capacity. At 11 cameras recording in 4K, the setup sustains 12 days of continuous footage before the oldest recordings cycle out.

## AI-powered detection — and the false positive problem

Unifi Protect ships with AI-based smart detection out of the box: vehicle detection, person detection, facial recognition, and license plate reading (LPR). Getting the hardware running is the easy part. Getting the detection to actually be useful is where the real work starts.

The first problem that surfaced was the LPR alerts. The cameras covering the parking lot were also picking up vehicles driving past on the main road — triggering alerts for every car that happened to pass the building. In a busy area, that generates a lot of noise very quickly. Noise is the enemy of effective monitoring; if every alert requires a manual check, the system stops being a force multiplier and starts being a burden.

The fix was adjusting the smart detection zones — Unifi Protect lets you draw a polygon on the camera's field of view and restrict AI detection to only that region. By tightening the zone to cover only the actual parking lot and excluding the road, the false positive rate dropped significantly. The cameras still captured the full scene for footage purposes, but alerts were only triggered by vehicles entering or moving within the designated area.

This is exactly the same problem a SOC analyst faces when tuning SIEM rules. A detection that fires too broadly generates alert fatigue. The answer in both cases is scope reduction — define precisely what you want to detect, exclude the noise sources, and iterate until the signal-to-noise ratio is actually useful.

## Case management in practice

Unifi Protect's case management feature lets you pull clips from multiple cameras, attach them to a single incident, add notes, and organize everything into a timeline. It's essentially an evidence management system built into the NVR.

We used it on the first day the system was fully operational. A disgruntled individual came into the building and confronted employees. With case management, I was able to pull footage from the front entrance camera showing the person arriving and the exit camera showing them leaving — bundled into a single case with timestamps and notes documenting the sequence of events.

That's an incident response workflow. Detection, scoping, evidence collection, timeline reconstruction — the structure is identical to what a SOC analyst does when working a security incident, just applied to a physical event rather than a digital one. The discipline of documenting what happened, when, and in what order matters just as much when the incident involves a physical confrontation as when it involves a compromised endpoint.

## Physical deployment challenges

One thing that doesn't show up in most design sheets: camera mounting angles. Several of the G6 cameras needed to cover areas where the ideal mounting point didn't give the right angle without an extension accessory. Finding that out during installation rather than planning is a time cost. The lesson is that site surveys matter — walking the space and mapping camera fields of view before ordering hardware saves time during deployment.

The 360 Pro was the most useful single camera in the deployment. One unit covers a wide area that would otherwise require two or three standard cameras, which simplifies both cabling and the number of devices to manage and maintain.

## On-site storage as a security decision

The decision to keep all footage on the RAID NVR on-site rather than using cloud storage wasn't just a cost decision — it was a deliberate security and privacy posture. Cloud-hosted surveillance footage creates a third-party data custody question: who has access, under what circumstances, and what happens if the cloud provider is breached?

On-site storage keeps the data entirely within the organization's control. The tradeoff is that if the NVR itself is physically compromised or destroyed in the same incident being investigated, the footage could be lost. RAID 5 addresses hardware failure but not physical destruction. For a higher-security deployment, off-site or cloud backup of critical footage would be the next layer — with appropriate access controls.

## What this maps to in security operations

Running this system day-to-day has direct parallels to SOC work:

**Detection tuning** is the same problem whether you're adjusting smart detection zones on a camera or refining a SIEM rule. Both require understanding what you're trying to detect, identifying the noise sources, and iterating until alerts are actionable.

**Alert fatigue is universal.** A surveillance system that generates hundreds of false positive vehicle alerts is useless. A SIEM that fires on every outbound connection is useless. The goal in both cases is high-fidelity alerting — fewer alerts that actually mean something.

**Evidence handling matters.** Case management in Unifi Protect imposes a structure on evidence collection that would hold up to scrutiny. The same discipline applies to incident documentation in a SOC — a timeline with attached artifacts and notes is more useful than a collection of screenshots.

**End-to-end ownership teaches you the whole system.** Deploying and maintaining a security system solo — hardware selection, configuration, tuning, incident response — forces you to understand every layer. There's no handoff to another team when something breaks. That breadth is uncomfortable sometimes, but it builds a more complete picture of how security systems actually work.

Physical and digital security aren't separate disciplines. They're the same discipline applied to different surfaces.
