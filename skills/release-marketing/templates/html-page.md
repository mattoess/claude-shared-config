# HTML Page Template Reference

When generating blog posts or landing pages, use this skeleton. Fill in the bracketed values.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- Enhanced Meta Tags -->
    <title>[PAGE_TITLE] | Revenue Growth Agent</title>
    <meta name="description" content="[META_DESCRIPTION_150_160_CHARS]" />
    <link rel="canonical" href="https://www.revenuegrowthagent.com/[SLUG]/" />

    <!-- Keywords for AI Discovery -->
    <meta name="keywords" content="[COMMA_SEPARATED_KEYWORDS]" />
    <meta name="author" content="Matt Oess" />
    <meta name="robots" content="index, follow, max-snippet:-1, max-image-preview:large" />

    <!-- Open Graph -->
    <meta property="og:type" content="[website|article]" />
    <meta property="og:title" content="[OG_TITLE]" />
    <meta property="og:description" content="[OG_DESCRIPTION]" />
    <meta property="og:url" content="https://www.revenuegrowthagent.com/[SLUG]/" />
    <meta property="og:site_name" content="Revenue Growth Agent" />
    <meta property="og:image" content="https://www.revenuegrowthagent.com/hero-video-poster.jpg" />

    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image" />
    <meta name="twitter:title" content="[TWITTER_TITLE]" />
    <meta name="twitter:description" content="[TWITTER_DESC_UNDER_200_CHARS]" />
    <meta name="twitter:image" content="https://www.revenuegrowthagent.com/hero-video-poster.jpg" />

    <link rel="icon" type="image/png" href="/FAVICON40x40.png" />

    <!-- Tailwind CSS - Production Build -->
    <link rel="stylesheet" href="/css/about-styles.min.css">

    <!-- Custom dropdown styles -->
    <style>
        .group:hover .group-hover\:opacity-100 {
            opacity: 1 !important;
        }
        .group:hover .group-hover\:visible {
            visibility: visible !important;
        }
    </style>

    <!-- Google Analytics -->
    <script async src="https://www.googletagmanager.com/gtag/js?id=G-F1BFC450CK"></script>
    <script>
      window.dataLayer = window.dataLayer || [];
      function gtag(){dataLayer.push(arguments);}
      gtag('js', new Date());
      gtag('config', 'G-F1BFC450CK');
    </script>
</head>
<body class="bg-white">

    <!-- Schema.org Markup -->
    <script type="application/ld+json">
    {
      "@context": "https://schema.org",
      "@graph": [
        {
          "@type": "Organization",
          "@id": "https://www.revenuegrowthagent.com/#organization",
          "name": "Revenue Growth Agent",
          "description": "AI-powered sales platform for meeting prep, discovery, and proposals",
          "url": "https://www.revenuegrowthagent.com",
          "logo": "https://www.revenuegrowthagent.com/logo.png",
          "sameAs": [
            "https://www.linkedin.com/company/revenuegrowthagent"
          ]
        },
        {
          "@type": "[WebPage|Article|BlogPosting]",
          "@id": "https://www.revenuegrowthagent.com/[SLUG]/#[webpage|article]",
          "url": "https://www.revenuegrowthagent.com/[SLUG]/",
          "name": "[SCHEMA_NAME]",
          "description": "[SCHEMA_DESCRIPTION]",
          "publisher": {
            "@id": "https://www.revenuegrowthagent.com/#organization"
          },
          "isPartOf": {
            "@type": "WebSite",
            "url": "https://www.revenuegrowthagent.com",
            "name": "Revenue Growth Agent"
          }
        }
      ]
    }
    </script>

    <!-- NAVIGATION: Copy nav from an existing page like about.html -->

    <!-- MAIN CONTENT -->
    <main>
      <!-- Hero Section -->
      <!-- Content Sections -->
      <!-- CTA Section -->
    </main>

    <!-- FOOTER: Copy footer from an existing page like about.html -->

</body>
</html>
```

## Notes

- Always read the latest `public/about.html` for the current nav and footer HTML
- Use Tailwind utility classes exclusively — no custom CSS except the dropdown styles above
- CTA buttons should use: `class="appearance-none px-8 py-4 bg-[#fe6601] text-white rounded-lg hover:bg-[#fea735] ..."`
- Link signup CTAs to `/?signup=true`
- Internal links: Use relative paths for other pages (e.g., `/about/`, `/ai-sales-platform-for-saas-companies/`)
- All images referenced must already exist in `public/` — do not invent image paths
