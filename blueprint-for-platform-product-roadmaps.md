---
layout: single
title: "Blueprint for Platform Product Roadmaps"
permalink: /blueprint-for-platform-product-roadmaps/
toc: true
toc_sticky: true
---

## A practical guide for people building platforms in 2024\.

By [Vik Chaudhary](https://www.linkedin.com/in/vikchaudhary) and [Sid Palani](https://www.linkedin.com/in/sidpalani)

Published on Tue Jul 9, 2024 on [LinkedIn](https://www.linkedin.com/feed/update/urn:li:activity:7216475133052432388?updateEntityUrn=urn%3Ali%3Afs_feedUpdate%3A%28V2%2Curn%3Ali%3Aactivity%3A7216475133052432388%29), [Neuronn on LinkedIn](https://www.linkedin.com/posts/neuronn_there-are-many-resources-for-product-managers-activity-7216440984052588545-Bko6?utm_source=share&utm_medium=member_desktop), [Substack](https://neuronn.substack.com/p/blueprint-for-platform-product-roadmaps?r=hhsh6&utm_campaign=post&utm_medium=web&triedRedirect=true) and [Google Slides](https://docs.google.com/presentation/d/113AWG-9jQnw_atpwovsRDxFco6JSIVHcN-h8qhk8DOI/edit?usp=sharing)

![Blueprint for Platform Product Roadmaps](/assets/img/cover-blueprint-for-platform-product-roadmaps.png "Blueprint for Platform Product Roadmaps")

# Our Motivation

There are many resources for product managers of consumer apps or B2B applications. However, we see a gap in PM resources for the unique challenges of building platforms, which serve as foundational tools for developers (e.g., AWS, Cohere, GitHub, Google Vertex AI, OpenAI GPT, Shopify, Stripe, and Twilio) and are becoming increasingly common in the AI era. This article aims to bridge that gap by providing a framework to outline a platform roadmap. It is intended for product managers, engineering leaders, CIOs, and CEOs thinking about building platforms, and uses an example framework to create strategy, articulate benefits, prioritize features, and deliver a roadmap for a platform product.

Our experience with developer and ML platform teams at Meta is that roadmaps often begin with goals like collaboration, code reuse, and efficiency. However, teams easily become fragmented, leading to frequent software updates, numerous API versions, legacy code use cases, incoherent user experiences, and growing technical debt. The problems that compound over time include:

* **Fragmented User Experience**. Developer experience is spread across many UX surfaces, which users struggle to navigate, relying on word-of-mouth guidance, and incomplete documentation hampers onboarding. At Meta, engineers encountered dozens of applications throughout the software lifecycle, when perhaps ten might have sufficed.

* **Software Management.** There is often no comprehensive view of all application dependencies like libraries, data stores, or clusters. Tracking signals such as QA status, deployments, alerts, and open tasks are done ad hoc. This raises a developer’s cognitive load and inefficiencies.

* **Unreliable Systems.** Automated dependency updates cause more production errors, leading to time-consuming debugging, crucial downtime incidents, and slow root cause analysis. 

# What Are Platform Products?

A software platform provides a foundation (e.g., environment, tools) that developers, either internal or external, use to build applications more easily than building and maintaining everything themselves.

Some renowned platforms for developers include **Stripe**, which powers website payments through easy-to-use APIs in familiar languages like JavaScript or Python. **Kubernetes** helps scale websites during peak times, such as Black Friday sales, by automatically adjusting resources, like servers, to meet demand without overpaying for unused resources. **Hugging Face** is an AI platform offering tools and models for natural language processing (NLP). For example, it can help a news service generate brief summaries from long articles, saving users time while keeping them informed.

![Examples of Platform Products](/assets/img/examples-of-platform-products.png "Examples of Platform Products")

# Framework

In our examples, we will address developers' workflow using a platform to build front-end applications, back-end services, or data management services. In doing so, we plan to answer the following questions:

1\. How do you set a platform's vision, strategy, and guiding principles?  
2\. How do you measure success?   
3\. How do you prioritize a platform roadmap?  
4\. How do you create a timeline for the roadmap?  
5\. What is the Go-to-Market plan that ensures adoption?

# 1\. How do you set a platform's vision, strategy, and guiding principles?

## Product Vision

A product vision is a statement that outlines a product's purpose, value, and long-term goals. Here’s an example of an internal developer platform at Meta that helps developers build and operate consumer apps like Facebook:

![Example Vision](/assets/img/example-vision.png "Examples of Platform Product Vision")

## Principles for Internal Platforms

When building a Platform for internal use, be driven by deep discovery, 10x problem-solving, adoption, and impact. Our guiding principles are:

1. Adopt a product mindset  
2. Focus on customers critical to growth  
3. Provide self-service capabilities

**1\. Adopt a product mindset**

* Internal developer tools should solve the most painful problems first.  
* Have an opinionated point of view on the tools offered and reduce UX surfaces.  
* Measure the success of accelerating software delivery and reducing time spent on incidents.

**2\. Focus on customers critical to growth**

* Segment developers to understand their impact on future growth: e.g. ML vs software engineers.  
* Understand each segment’s critical needs to build the platform roadmap.  
* Prioritize features based on requirements, effort, and measurable impact on business.

**3\. Provide self-service capabilities**

* Hide complexity: automate feature delivery, testing, performance, and security requirements in a simple UX.  
* Simplify deployment: Clear documentation, fewer services, common configuration templates / smart defaults, and standardized libraries for observability and workload deployment.  
* Deploy customized onboarding tools based on developer personas.

## Benefits of a Developer Platform

What is the rationale for building a developer platform? A platform allows a company to provide its core offerings in a way that can easily adapt and scale into a large ecosystem of customers and partners. It benefits the ecosystem because it creates a common language for building and adopting core features, making development easier and increasing velocity, leverage, and impact for the business. 

![Platform Benefits](/assets/img/platform-benefits.png "Platform Benefits")

The best-in-class tooling from each development team can be centralized in the engineering platform for all other teams to benefit from. In addition, having a robust, well-documented engineering platform can expedite the time from hire to first pull request, increasing the productivity of new or transferred developers.

## Evergreen Priorities

While every platform is unique, it shares a set of “always-on” priorities that ground a roadmap: developer experience, delivery, and reliability. Developers are not only the core users but also the force multipliers for the platform’s impact and scale. These priorities address core needs regardless of specific use cases and provide helpful constraints to manage the numerous possible directions platforms can take.

![Evergreen Priorities ](/assets/img/evergreen-priorities.png "Evergreen Priorities")

## Considerations for AI / ML Platforms

Platforms that employ artificial intelligence and machine learning features have specific important nuances worth highlighting. It is important that priorities account for these elements:

1. **Data Management.** Data is the lifeblood of AI models–it drives the training, fine-tuning, and evaluation of models put into use. Therefore, a scalable ML platform needs to consider specifically how developers can easily and securely store, retrieve, and label training and evaluation data that is used in model development. 

2. **Cost (Compute) Management.** As developers are given more control over training and fine-tuning models for specific use cases, the platform needs to account for how to share and allocate training costs and provide control to the developers to manage within their budgets. This is especially true today while computation cost is a major barrier to leveraging AI.

3. **Steerability.** As use cases for ML continue to expand beyond largely creative use cases into regulated industries (e.g. healthcare, finance) with a greater degree for harm, the ability for developers to understand and interrogate why models make predictions they do becomes imperative–both for rapid product improvement and societal trust. A long-term consideration for ML platforms will be investment in features that provide transparency to developers, with interpretability and explainability.

4. **Success Measures.** A measurement framework (discussed below) will need to integrate common ML model performance metrics such as accuracy, precision, and recall that are often upstream of core product metrics but offer another area that can either help or hurt a developer’s experience of the platform.

# 2\. How do you measure success?

## Setting Goals

Measuring success for platform products is especially challenging because they are removed from end use cases. As such, we focus on the primary developer users and use a broad set of dimensions to encompass how well the platform delivers the value promised. A framework commonly used at Google is the QUANTS framework, which we describe below.  

![Setting Goals for Platforms](/assets/img/setting-goals-for-platforms.png "Setting Goals for Platforms")

## Example: Code Search metrics using QUANTS

For example, we’ll describe a hypothetical feature called code search (a search engine dedicated to finding specific sections of program code) and break down how we measure success using the QUANTS framework.

![Code Search Metrics Using QUANTS Framework](/assets/img/code-search-metrics-using-quants-framework.png "Code Search Metrics Using QUANTS Framework")

Once developers adopt and use your platform tools, they will depend on their reliability. Within a company, this responsibility would fall to a developer operations organization. We recommend measuring reliability with a separate framework to minimize conflicts with other priorities.  

![Measuring Reliability](/assets/img/measuring-reliability.png "Measuring Reliability")

# 3\. How do you prioritize a platform roadmap?

## Common Pitfalls of Platform PM

A Platform PM will encounter challenges when creating and evangelizing a product roadmap. These include convincing product teams to migrate from ad-hoc tooling to centralized engineering platforms, introducing new tools to avoid further fragmentation and complexity, and setting proxy metrics when top-line metrics can be costly or difficult to measure regularly.

When platform teams are first formed, PMs face particularly large challenges in convincing client teams to adopt a new platform over the ad-hoc solutions they have developed. The latter often leads to tool fragmentation and dependencies on multiple versions of software libraries, which result in more reliability issues. When this happens, more time is lost to debugging problems, often during a time-sensitive phase when a severe event like downtime occurs. Collecting data on reliability incidents and their root causes, and doing qualitative research to understand the technical factors that reduce developer productivity are ways that PMs can build a business case for a more robust platform. Some ways that PMs can address adoption if your platform is new and not feature-rich:

1. In a metric-driven culture, a PM could define a clear and convincing narrative about how the platform will be accretive to one of the client team's goal metrics.

2. For early adopters, the platform team can consider supporting the implementation alongside the client team and should plan resources appropriately.

3. When possible, the platform team can propose experiments to demonstrably prove success stories instead of testimonials only.

4. Assess whether we can reuse, extend, and deprecate existing tooling to minimize additional fragmentation.

## Technique: Overlay Industry Research

Over twenty years in the developer platform space have shown us that developers face many problems, including change management due to multiple software versions, fragmented user experience from too many tools, and hard-to-find or outdated documentation. Creating a roadmap to address these issues relies on judgment from product management and engineering leaders. Our experience indicates that these decisions are often subjective and influenced by the most prominent voices or arbitrary consensus-building.

Products and platforms share a common challenge: there are usually more problems than time to address them all, making prioritization crucial. However, with a diverse developer user base and specific feedback, prioritizing becomes particularly challenging and often relies on qualitative judgment by product and engineering leaders. To stay data-driven, mapping credible industry research against user problems provides an objective prioritization filter (see below). 

In this example, we highlight (in yellow) specific problems to solve. To narrow down the many potential issues, we mapped them to findings from industry research, using Harness’ [State of the Developer Experience Report 2024](https://www.harness.io/state-of-developer-experience).

![Harness State of Developer Experience Report 2024](/assets/img/harness-state-of-developer-experience.png "Harness State of Developer Experience Report 2024")

## Theme by Priority Problems

By anchoring on problems sized by research, we can then embark on the standard product exercise of brainstorming solutions and grouping them in themes against each problem.  

![Grouping Into Themes](/assets/img/grouping-into-themes.png "Grouping Into Themes")

## What new directions should we explore?

In addition to using data to create a roadmap, it’s useful to use our experience and judgment to explore new long-term directions and/or new business expansion initiatives. This typically requires combining company-level strategy and important market trends that the company must address in order to succeed.

For example, Meta’s strategic directions are to (a) transform its products from mobile and web to virtual reality (VR) headsets, (b) optimize the delivery of advertising and reverse ad revenue loss from systems issues, and (c) make its 50,000+ developers more productive. With these objectives in mind, we might suggest some new directions for Meta’s developer platforms, below.

![Directions for Meta Developer Platforms](/assets/img/directions-for-meta-developer-platforms.png "Directions for Meta Developer Platforms")

# 4\. How do you create a timeline for the roadmap?

Recall that **Leverage**, **Velocity**, and **Impact** are the benefits of a Platform. An effective way to create a roadmap timeline is to map features on a 2x2 matrix. Here we use Leverage and Velocity as the X and Y coordinates, placing features into four quadrants. Features in the top right quadrant are prioritized for early delivery, while those in the bottom left are delivered last.

There may be exceptions to the ordering. Features that deliver substantial business impact (Impact being our third benefit), might be accelerated in the timeline. Additionally, use judgment when necessary. For example, improving documentation for internal tools may have low Leverage and moderate Velocity but it could still be prioritized for earlier delivery due to its common-sense value.

![Product Roadmap Timeline](/assets/img/product-roadmap-timeline.png "Product Roadmap Timeline")

# 5\. What is the Go-to-Market plan that ensures adoption?

We usually conclude a Platform roadmap with specific steps to ensure the adoption of new capabilities. It is crucial that the teams responsible for execution—usually sales, marketing, and customer service—explicitly agree on this plan.

![Product Roadmap Plan](/assets/img/product-roadmap-plan.png "Product Roadmap Plan")

# Conclusion

This article assumes that the product leader has a strong understanding of the product roadmap process. It is meant to highlight the main differences in building platforms as compared to products. We hope this article will be a helpful starting resource for building platforms and we would love to hear your feedback and suggestions on it, other resources, and future articles.

Finally, many thanks to our Neuronn AI colleagues and [Alanna Viega](https://www.linkedin.com/in/alanna-veiga/) for reviewing and providing feedback on this article.

# About Neuronn AI

Neuronn AI is a professional advisory group rooted in prominent Artificial Intelligence companies such as Meta. Leveraging extensive expertise in Data Science & Data Engineering, AI Product Management, Market Research, and Strategic and Brand Marketing, we offer fractional assistance to startups looking to launch AI-driven product ideas and provide consulting services to enterprises seeking to implement AI to accelerate their outcomes. Neuronn AI's advisors include [Alex Kalinin](https://www.linkedin.com/in/alex-kalinin-seattle), [Carlos Romero](https://www.linkedin.com/in/carlos-f-romero/), [Enrique Ortiz](https://www.linkedin.com/in/enriqueortiz), [Lacey Olsen](https://www.linkedin.com/in/laceyolsen), [Norman Lee](https://www.linkedin.com/in/normanhlee/), [Owen Nwanze-Obaseki](https://www.linkedin.com/in/thebritishtexan/), [Rick Gupta](https://www.linkedin.com/in/rickgupta/), [Seda Palaz Pazarbasi](https://www.linkedin.com/in/sedapalazpazarbasi/), [Sid Palani](https://www.linkedin.com/in/sidpalani) and [Vik Chaudhary](https://www.linkedin.com/in/vikchaudhary).



