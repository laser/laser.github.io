# Erin "Laser" Swenson-Healey
California, USA | github.com/laser | ICQ: 2176689

## Summary

I'm an 1990's-forged nerd (think 2600 Magazine, IPX emulators, Rifts, DikuMUDs, and the turbo button) and lifelong technologist with a passion for solving human problems with computer software. I have 25 years of experience building mission-critical systems from roboticized manufacturing cells to cryptocurrencies and a track record of delivering foundational infrastructure at companies ranging from startups to public enterprises. I have deep expertise in system architecture, platform engineering, and turning complex technical challenges into production-ready software.

## Experience

### Hadrian | Staff Software Engineer, Robotics
*March 2023 - Present*

**Project 1: Automated Manufacturing Cell ($10MM-$15MM hardware per cell)**

- Architected and implemented control system for first automated CNC cell at Hadrian (12 five-axis machines, 2 KUKA robots)
- Achieved 50% cost reduction ($7.2MM per cell) with 99.99% uptime for 24/7 "lights-out" production
- Designed on-call rotation and started "zero defects" culture where every ERROR-level/off-nominal system behavior log triggers pages to on-call duty
- Delivered 110K lines of TypeScript (60K implementation, 50K tests) with test suite robust enough to deploy straight to production w/out pre-production environment or simulation
- Built fault-tolerant system where any component can fail without stopping production
- Built blessed-based TUI for administering/troubleshooting control system

**Project 2: libcontrol Platform Product**

- Generalized linear rail control system into reusable platform adopted by 3 teams (CNC, CMM, tool-building)
- Reduced control system development time from ~7 months to 3-4 weeks
- Provides data persistence, task processing, alarm management, telemetry, resource locking, and admin TUI

*Technologies: TypeScript, Node.js, Go, SQLite, Python, PostgreSQL, Datadog, ROS 2, RoboDK*

### Stitch Fix | Principal Platform Developer
*November 2021 - March 2023*

- Led platform team managing RabbitMQ and Kafka clusters for 250+ engineers across 75 applications
- Architected and executed company-wide migration from RabbitMQ to Kafka, saving **$50K/month**
- Built SFIX-specific, messaging-focused libraries in Ruby and Go
- Optimized librdkafka performance through configuration tuning and FFI work
- Improved platform reliability by standardizing on Confluent Cloud, eliminating frequent CloudAMQP outages

*Technologies: Ruby, Go, Kafka, RabbitMQ, librdkafka, Confluent Cloud, CloudAMQP, Sidekiq*

### Carbon Five | Principal Developer
*March 2013 - November 2021*

**Protocol Labs - Filecoin Implementation ($1B+ market cap)**

- One of 10 founding engineers building initial Filecoin node implementation
- Built proof-of-storage scheduler in Rust, managing cryptographic proof generation and customer data persistence
- Sole developer of FFI bindings between Go node and Rust proof system - no memory leaks since launch
- Bridged gap between cryptographers and engineers, translating theoretical concepts to production code
- Set up CI/CD, cloud testing infrastructure, and mentored team on testing and quality practices
- Multi-year project w/very successful launch

**Other Projects**

- Gap Inc. - Forecasting and inventory management systems
- First Republic Bank - Custom CRM + Jenkins infrastructure
- Fandango - Reverse proxy microservice aggregation + caching system
- Red Bull - Event management system
- NPFL - Scoring system used to power the Jumbotron

*Technologies: Go, Rust, TypeScript, RDBMS, NoSQL*

## Open Source Work

I have made significant contributions to several high-use open-source projects over the years. If you want to get an idea for how I write software, these links would be a good place to start:

- [https://github.com/filecoin-project/filecoin-ffi](https://github.com/filecoin-project/filecoin-ffi)
- [https://github.com/filecoin-project/rust-fil-sector-builder](https://github.com/filecoin-project/rust-fil-sector-builder)
- [https://github.com/filecoin-project/venus](https://github.com/filecoin-project/venus)
- [https://github.com/laser/cloudformation-fargate-codepipeline-ecs-refarch](https://github.com/laser/cloudformation-fargate-codepipeline-ecs-refarch)
- [https://github.com/filecoin-project/rust-fil-proofs](https://github.com/filecoin-project/rust-fil-proofs)

## Talks and Blog Posts

I have given a number of talks over the last few years on a variety of technical subjects. This is a non-exhaustive list, but gives some idea of the things that I've been thinking about over the last decade or so. If you'd like to see samples of architectural documents that I've produced in the past, please feel free to ask.

- [How I Use Claude Code](https://docs.google.com/presentation/d/13iNT_8PTYlNglqOsl-h_RKkitnpNIJwTR2UigGfzEF8/edit?usp=sharing) | slides
- [Noodling w/Generics and Recursive Interfaces, in Go](https://erinswensonhealey.com/blog/20230131-noodling-go-generics-recursive-interfaces.md.html) | blog
- [A Beginner’s Guide to Exceptions in Haskell](https://www.youtube.com/watch?v=hC7hwEQtdnE) | video | Santa Monica Haskell User Group
- [Haste: Full-Stack Haskell for Non-PhD Candidates](https://www.youtube.com/watch?v=3v03NFcyvzc) | video | Strange Loop
- [A n00b's Guide to Writing a Network Server (in Ruby)](https://docs.google.com/presentation/d/1ionEJfX2EHAjjV4ZzDOfaFLG8uKvi3geanq8LAxS8fg/edit?usp=sharing) | slides | Repeat.io Community Night
- [TCP/IPv4 Basics: ARP, IP Routing, TCP, and NAT](https://slides.com/laser/tcp-ip-basics-15) | slides | Carbon Five Summit
- [Hanging Up on Callbacks: Generators in ES6](https://www.youtube.com/watch?v=kH_WuLbb-8A) | video | HTML5 DevConf
- [Merkle Tree: Building Block of Decentralized Web](https://www.youtube.com/watch?v=HdGpG0kcEGU) | video | Chadev
- [An Introduction to ADTs and Structural Pattern Matching in TypeScript](http://erinswensonhealey.com/blog/20180108-structural-adt-typescript.md.html) | blog
- [The JavaScript Event Loop: Explained](http://erinswensonhealey.com/blog/20131027-event-loop-explained.md.html) | blog

## Education

**University of Washington** - Bachelor of Arts

**Universidad Nacional Autónoma de México** - Extended study (CEFR B2 Spanish)
