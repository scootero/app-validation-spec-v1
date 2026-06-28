Phase 1: App Validation Specification

Documentation & Contract Plan

Overview

Establish the app-validation-spec repository as the official documentation-and-contract foundation for an automated app validation system.

This repo defines how every future app idea should be packaged so automation tools like n8n, Cursor, Codex, Vercel, Meta Ads, analytics dashboards, and landing page templates can consume the same standardized input.

Phase 1 is documentation and contract only.

Do not build:

* validator CLI
* n8n workflows
* landing page generator
* Vercel deployment logic
* Facebook ad publishing
* dashboard

⸻

1. Current Repo Structure

The current folder should be reused:

app-validation-spec/
├── APP_PACKAGE_SPEC.md
├── CHANGELOG.md
├── README.md
├── docs/
│   ├── design-philosophy.md
│   ├── n8n-integration-notes.md
│   ├── naming-conventions.md
│   ├── versioning.md
│   └── workflow.md
├── examples/
│   ├── full-app/
│   │   ├── app.json
│   │   ├── copy/
│   │   │   ├── faq.md
│   │   │   ├── features.md
│   │   │   └── hero.md
│   │   ├── media/
│   │   │   └── README.md
│   │   ├── mockup/
│   │   │   └── README.md
│   │   └── README.md
│   └── minimal-app/
│       ├── app.json
│       └── README.md
├── schemas/
│   └── app.schema.json
└── templates/
    ├── app.json
    └── copy/
        ├── faq.md
        ├── features.md
        └── hero.md

This is the correct Phase 1 structure.

Do not delete it.

Update it.

⸻

2. Project Purpose

The system will eventually allow a user to create a standardized app package in Google Drive.

n8n will later:

Read App Package
↓
Validate app.json and required files
↓
Deploy mockup app
↓
Generate landing page config
↓
Deploy landing page to Vercel
↓
Create Facebook ad assets
↓
Track emails and Buy Now clicks
↓
Feed metrics into analytics dashboard
↓
Decide whether to build, kill, pause, or continue validating

The App Package is the source of truth.

⸻

3. Core Design Philosophy

The spec should be:

* human-editable
* AI-generatable
* n8n-friendly
* JSON-schema-validatable
* versioned
* flexible across many app categories
* strict enough for reliable automation
* reusable for landing pages, ads, dashboards, App Store metadata, analytics, and mockup deployment

The repo should define the contract.

The validator, workflows, and generators come later.

⸻

4. Updated app.json Top-Level Structure

The improved top-level schema should include:

{
  "specVersion": "1.0.0",
  "status": "draft",
  "appId": "focus-timer-pro",
  "identity": {},
  "audience": {},
  "experiment": {},
  "commerce": {},
  "branding": {},
  "mockup": {},
  "media": {},
  "landingPage": {},
  "ads": {},
  "tracking": {},
  "analytics": {},
  "deployment": {},
  "appStore": {}
}

⸻

5. Required Root Fields

Required:

specVersion
status
appId
identity
audience
experiment
commerce
landingPage

Optional but strongly supported:

branding
mockup
media
ads
tracking
analytics
deployment
appStore

⸻

6. New status Field

Add a root-level status field.

Allowed values:

draft
ready
validating
paused
winner
killed
built

Purpose:

* helps n8n know what stage the app package is in
* helps dashboards categorize experiments
* helps avoid accidentally launching incomplete packages

Example:

"status": "draft"

⸻

7. New experiment Section

Add top-level experiment.

This is important because the real product is not just the app — it is the validation test.

Example:

"experiment": {
  "experimentName": "Focus Timer Paid Download Test",
  "hypothesis": "Busy iPhone users will click a paid App Store CTA for a simple focus timer if the landing page clearly shows the benefit.",
  "successCriteria": {
    "minimumEmailConversionRate": 0.08,
    "minimumBuyNowClickRate": 0.03,
    "minimumVisitors": 300
  },
  "testBudget": {
    "currency": "USD",
    "amount": 100
  },
  "decisionRules": {
    "winner": "If Buy Now click rate is above 3% after at least 300 visitors.",
    "kill": "If Buy Now click rate is below 1% after 300 visitors.",
    "iterate": "If emails are strong but Buy Now clicks are weak."
  }
}

Purpose:

* defines what success means before running ads
* makes dashboard decisions easier
* prevents vague validation
* helps compare app ideas objectively

⸻

8. Updated mockup Section

The mockup section should support both source-project information and deployed preview information.

Example:

"mockup": {
  "type": "interactive",
  "sourcePath": "mockup/",
  "framework": "react-vite",
  "entryPoint": "src/App.jsx",
  "installCommand": "npm install",
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "deployCommand": null,
  "previewUrl": null,
  "notes": "This mockup controls its own iPhone-style dimensions and should be embedded by URL."
}

Purpose:

* supports your current JSX/Vite mockup projects
* lets n8n know where the mockup lives
* lets n8n deploy the mockup first
* gives landing pages the final iframe URL later

⸻

9. New deployment Section

Add top-level deployment.

Example:

"deployment": {
  "mockupUrl": null,
  "landingPageUrl": null,
  "githubRepoUrl": null,
  "vercelProjectId": null,
  "vercelDeploymentUrl": null,
  "lastDeployedAt": null
}

Purpose:

* stores generated URLs and platform IDs
* supports n8n/Vercel workflow state
* helps dashboards link back to live experiments
* prevents deployment info from being scattered across workflow history only

⸻

10. New Optional appStore Section

Reserve an optional appStore section.

Example:

"appStore": {
  "appStoreUrl": null,
  "subtitle": "",
  "keywords": [],
  "promotionalText": "",
  "description": "",
  "supportUrl": "",
  "privacyPolicyUrl": ""
}

Purpose:

* future App Store listing generation
* future App Store button behavior
* future SEO and paid ad consistency
* not required for Phase 1 validation

⸻

11. identity Section

Example:

"identity": {
  "appName": "Focus Timer Pro",
  "tagline": "A cleaner way to focus on your iPhone.",
  "description": "A simple iOS-style focus timer designed to help users stay productive without clutter.",
  "category": "productivity",
  "categoryLabel": "Productivity",
  "platform": "ios",
  "tags": ["focus", "timer", "productivity", "iphone"]
}

Notes:

* appId is stable and should not change.
* appName can change.
* category should support enums plus "other".
* categoryLabel allows human-readable flexibility.

⸻

12. audience Section

Example:

"audience": {
  "primary": "iPhone users who want a simple way to stay focused.",
  "secondary": "Students, remote workers, and creators.",
  "painPoints": [
    "Most productivity apps are too bloated.",
    "People want focus tools that are fast and simple."
  ],
  "personas": [
    {
      "name": "Busy Remote Worker",
      "summary": "Needs short focus sessions without a complicated productivity system."
    }
  ]
}

⸻

13. commerce Section

Example:

"commerce": {
  "pricing": {
    "model": "paid",
    "currency": "USD",
    "amount": 4.99,
    "period": null,
    "trialDays": null
  },
  "cta": {
    "primaryText": "Get it on the App Store",
    "secondaryText": "See it in action",
    "buyNowText": "Buy on the App Store",
    "waitlistText": "Notify Me"
  }
}

Important:

CTA text must be configurable.

The system should not hardcode:

Early Access
Coming Soon
Buy Now
Get Started

Those should all come from config.

⸻

14. branding Section

Example:

"branding": {
  "theme": {
    "mode": "light",
    "style": "liquid-glass",
    "primaryColor": "#6D5DFB",
    "accentColor": "#A855F7",
    "fontFamily": "system"
  },
  "style": {
    "tone": "professional",
    "styleNotes": "Apple-inspired, premium, sleek, modern, minimal."
  }
}

Supported styles should include:

liquid-glass
apple-light
apple-dark
aurora
black-titanium
minimal-white
custom

⸻

15. media Section

Example:

"media": {
  "icon": {
    "path": "media/icon.png",
    "alt": "Focus Timer Pro app icon"
  },
  "logo": {
    "path": "media/logo.png",
    "alt": "Focus Timer Pro logo"
  },
  "ogImage": {
    "path": "media/og-image.png",
    "alt": "Focus Timer Pro preview image"
  },
  "screenshots": [
    {
      "path": "media/screenshots/01-home.png",
      "alt": "Focus timer home screen",
      "title": "Start focused sessions",
      "description": "Launch a timer in seconds."
    },
    {
      "path": "media/screenshots/02-stats.png",
      "alt": "Focus stats screen",
      "title": "Track progress",
      "description": "See daily focus streaks and session history."
    }
  ],
  "demoVideo": null
}

Important:

* do not assume exactly 3 or 4 screenshots
* screenshots should be dynamic
* each screenshot can have title, description, alt text, and path

⸻

16. landingPage Section

Example:

"landingPage": {
  "slug": "focus-timer-pro",
  "seo": {
    "title": "Focus Timer Pro - A Cleaner Way to Focus",
    "description": "A simple iPhone focus timer for people who want less clutter and more productivity.",
    "keywords": ["focus timer", "productivity app", "iphone timer"]
  },
  "sections": [
    {
      "id": "hero",
      "enabled": true,
      "source": "file",
      "file": "copy/hero.md"
    },
    {
      "id": "features",
      "enabled": true,
      "source": "file",
      "file": "copy/features.md"
    },
    {
      "id": "screenshots",
      "enabled": true,
      "source": "media"
    },
    {
      "id": "pricing",
      "enabled": true,
      "source": "inline"
    },
    {
      "id": "faq",
      "enabled": false,
      "source": "file",
      "file": "copy/faq.md"
    }
  ]
}

Important:

* sections should be ordered
* sections should be enable/disable configurable
* sections should support inline copy or file-based markdown
* this allows different app landing pages without code changes

⸻

17. ads Section

Example:

"ads": {
  "campaignName": "Focus Timer Pro Validation Test",
  "objective": "traffic",
  "platforms": ["facebook", "instagram"],
  "headlines": [
    "A cleaner way to focus on your iPhone",
    "Simple focus sessions without clutter",
    "Stay productive with a beautiful timer app"
  ],
  "primaryTexts": [
    "Most productivity apps are too complicated. Focus Timer Pro keeps it simple.",
    "Try a cleaner focus timer designed for iPhone users."
  ],
  "descriptions": [
    "Validate your interest today.",
    "Simple, focused, and built for iPhone."
  ],
  "callToAction": "LEARN_MORE",
  "utmTemplate": {
    "source": "facebook",
    "medium": "paid_social",
    "campaign": "{{appId}}-validation"
  }
}

⸻

18. tracking Section

Example:

"tracking": {
  "webhooks": {
    "validationComplete": null,
    "deployComplete": null,
    "emailCaptured": null,
    "buyNowClicked": null
  },
  "events": [
    {
      "name": "email_captured",
      "description": "User submitted email on landing page."
    },
    {
      "name": "buy_now_clicked",
      "description": "User entered email and clicked the Buy Now/App Store CTA."
    }
  ]
}

Important metadata to eventually collect:

email
appId
experimentId
eventName
price
pageUrl
referrer
utm_source
utm_medium
utm_campaign
utm_content
timestamp
anonymousVisitorId
sessionId
device/browser info

⸻

19. analytics Section

Example:

"analytics": {
  "projectId": "focus-timer-pro",
  "experimentId": "focus-timer-paid-test-001",
  "funnelName": "paid-traffic-validation",
  "customDimensions": {
    "category": "productivity",
    "platform": "ios"
  }
}

Purpose:

* dashboard grouping
* experiment comparison
* cross-channel reporting
* metrics attribution

⸻

20. Minimal App Package

Smallest useful valid package:

minimal-app/
├── app.json
└── README.md

Characteristics:

* inline landing page copy
* no media required
* no mockup required
* no ads required
* no webhook URLs required
* enough to render a simple validation page

⸻

21. Full App Package

Complete automation-ready package:

full-app/
├── app.json
├── copy/
│   ├── hero.md
│   ├── features.md
│   └── faq.md
├── media/
│   ├── icon.png
│   ├── logo.png
│   ├── og-image.png
│   └── screenshots/
│       ├── 01-home.png
│       ├── 02-feature.png
│       └── 03-pricing.png
├── mockup/
│   ├── package.json
│   ├── src/
│   │   ├── App.jsx
│   │   └── main.jsx
│   └── README.md
└── README.md

For Phase 1 examples, real binaries and mockup code are not required. Placeholder READMEs are acceptable.

⸻

22. Schema Strategy

Use JSON Schema Draft 2020-12.

Schema should:

* require core fields
* define enums for stable values
* allow optional advanced sections
* reject unknown fields where stable
* allow extension-friendly areas like customDimensions
* include descriptions for fields
* prepare for a future validator CLI

Use:

"$schema": "https://json-schema.org/draft/2020-12/schema"

⸻

23. Versioning Strategy

Every app.json must include:

"specVersion": "1.0.0"

Rules:

* Patch version: wording/docs/schema clarification without breaking packages
* Minor version: backward-compatible new optional fields
* Major version: breaking schema change

Docs should explain:

* app package version vs repo version
* migration expectations
* how n8n should handle old versions later

⸻

24. n8n Integration Notes

docs/n8n-integration-notes.md should explain:

* n8n should read app.json first
* n8n should treat app.json as the source of truth
* n8n should not parse README as trusted config
* n8n may read markdown files referenced by landingPage.sections
* n8n should validate package before deployment
* n8n should deploy mockup before landing page
* n8n should write deployment URLs back into deployment
* n8n should insert webhook URLs into tracking.webhooks
* n8n should fail early if required files are missing

⸻

25. Design Tradeoffs

JSON vs Markdown

Use JSON for structure.

Use Markdown for long copy.

Reason:

* JSON is machine-readable
* Markdown is human-friendly
* n8n can parse JSON reliably
* AI can generate both

App Package as CMS

The Google Drive folder should behave like a simple CMS.

Each app package contains everything needed to validate that idea.

Source of Truth

app.json is the source of truth.

README files are for humans only.

AI Role

AI should help generate or improve content, but deterministic workflows should consume structured fields.

⸻

26. Phase 1 Implementation Steps

Step A — Contract Core

Update or create:

README.md
APP_PACKAGE_SPEC.md
schemas/app.schema.json
templates/app.json
CHANGELOG.md

Step A should include all improved schema sections:

status
experiment
deployment
appStore
updated mockup structure

Step B — Templates and Examples

Update or create:

templates/copy/hero.md
templates/copy/features.md
templates/copy/faq.md
examples/minimal-app/
examples/full-app/

Examples must reflect the improved schema.

Step C — Supporting Docs

Update or create:

docs/design-philosophy.md
docs/workflow.md
docs/n8n-integration-notes.md
docs/naming-conventions.md
docs/versioning.md

⸻

27. Out of Scope for Phase 1

Do not build yet:

validator CLI
n8n workflows
landing page generator
Vercel deployment scripts
Facebook ad publisher
dashboard
real mockup deployment
real image assets

⸻

28. Cursor Implementation Instruction

Cursor should update the existing repo, not create a new nested folder.

It should preserve current files where possible, but update them to match this improved Phase 1 spec.

Start with Step A only unless explicitly told to continue.

After Step A, summarize:

files updated
schema sections added
important assumptions
what remains for Step B and Step C