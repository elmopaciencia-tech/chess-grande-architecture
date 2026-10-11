# System design

## Application boundaries

WordPress handles content, administration and request routing. Custom modules extend that foundation with booking and registration workflows. These modules share hosting and database infrastructure, so a plugin boundary is an organizational boundary rather than an independent service deployment.

```mermaid
%%{init: {"theme": "base", "themeVariables": {"background": "#ffffff", "fontFamily": "DM Sans, Avenir Next, Arial, sans-serif", "primaryColor": "#eafcff", "primaryTextColor": "#212f5f", "primaryBorderColor": "#212f5f", "secondaryColor": "#fff5fb", "secondaryTextColor": "#212f5f", "secondaryBorderColor": "#bd176f", "tertiaryColor": "#ffffff", "tertiaryTextColor": "#212f5f", "tertiaryBorderColor": "#212f5f", "lineColor": "#212f5f", "textColor": "#212f5f", "mainBkg": "#eafcff", "nodeBorder": "#212f5f", "clusterBkg": "#ffffff", "clusterBorder": "#1bc4df", "edgeLabelBackground": "#ffffff", "actorBkg": "#eafcff", "actorBorder": "#212f5f", "actorTextColor": "#212f5f", "actorLineColor": "#212f5f", "signalColor": "#212f5f", "signalTextColor": "#212f5f", "noteBkgColor": "#fff5fb", "noteBorderColor": "#bd176f", "noteTextColor": "#212f5f", "labelBoxBkgColor": "#fff5fb", "labelBoxBorderColor": "#bd176f", "labelTextColor": "#212f5f", "sequenceNumberColor": "#ffffff"}}}%%
flowchart TB
    families[Families and players] --> ui[WordPress pages and interactive interfaces]
    staff[Staff] --> admin[WordPress administration]
    subgraph app[Shared WordPress application]
      ui --> booking[Courses and holiday bookings]
      ui --> lessons[Private lessons]
      ui --> tournaments[Tournament registration]
      ui --> meetups[Free meetup requests]
      ui --> shop[Merchandise orders]
      ui --> quiz[React chess assessment]
      quiz -->|Result summary via browser event| booking
      admin --> booking
      admin --> tournaments
      admin --> meetups
      booking --> credits[Shared assessment credits]
      booking --> db[(WordPress database)]
      lessons --> db
      tournaments --> db
      meetups --> db
      shop --> db
      credits --> db
    end
    booking <--> stripe[Stripe]
    lessons <--> stripe
    tournaments <--> stripe
    shop <--> hitpay[HitPay]
    tournaments --> mail[Email delivery]
    meetups --> mail
    classDef external fill:#fff5fb,stroke:#bd176f,color:#212f5f;
    class staff,stripe,hitpay,mail external;
```

Arrows show the principal relationships, not every administration or notification route. Payment-provider connections include outbound checkout creation and incoming payment events. The free-meetup workflow has no payment-provider dependency.

## Data ownership

| Domain | Main records | Design purpose |
| --- | --- | --- |
| Editorial content | WordPress pages, posts, media and configuration | Keep publishing in familiar staff tools |
| Courses and holidays | Terms, courses, slots, requests and payment-event records | Model schedules and bookings explicitly |
| Tournaments | Event posts/configuration, registrations and payment-event records | Connect editable event details to traceable registrations |
| Meetups | Event content/state, parents, children, sessions, requests, audit and email outbox | Support family access, staff decisions and deferred delivery |
| Merchandise | Product content, variants, orders and order items | Separate product presentation from stock and order state |
| Assessment credits | Entitlements and reservation/redemption state | Coordinate one-time credits across participating booking modules |

These are logical record groups, not a database schema export. The application combines WordPress content storage with domain-specific tables. It does not require every workflow to fit into a generic post or a single universal order model.

## Frontend composition

The site combines theme and page-builder presentation with custom templates, shortcodes and frontend assets. The assessment uses React, react-chessboard and chess.js, bundled with Vite for WordPress to serve. The rest of the application does not need to become a React single-page app to support that interaction.

The assessment publishes a browser event containing the result summary. The course-booking interface listens for it and includes the summary when a visitor submits an assessment request. Browser-computed scores are advisory input, not a trusted certification of skill.

## Deployment and configuration

The project uses Local for development and documents WP Engine as its production host. Payment credentials belong in protected server configuration, separate from editable WordPress content and exported site data. Production webhook signing secrets are associated with their own handlers; local development listeners have separate configuration.

Server paths, account identifiers, endpoint inventories and secret names are excluded from this documentation.
