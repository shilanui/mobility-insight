Go to this repo
https://github.com/shilanui/mobility-frontend

How to Run (Need to setup database & seed test data from backend repository first)

- npm install
- npm run dev
- open brower with url "http://localhost:3000/fleet"

Framework & Libs

NextJS

- I use NextJS because it provides a fast development experience, built-in file based routing, and strong community support, which improves productivity and maintainability. However, SEO is not a primary concern since the system is designed as an authenticated multi tenant dashboard platform, where users access the application through login rather than public search engines.
- Also NextJS supports Server-Side Rendering (SSR), which is beneficial for pages that require frequent data fetching and heavy data presentation (Dashboard), improving both performance and user experience.

MUI (Material UI)

- I use MUI (Material UI) as the primary UI component library because it provides a flexible styling system through the sx prop, allowing fast and consistent customization. It also helps us avoid building a custom component system from scratch, which significantly reduces development time and ensures UI consistency across the application.

ApexCharts

- I use ApexCharts for data visualization because it provides interactive, responsive, and high-performance charts suitable for dashboard and analytics use cases. It supports multiple chart types and integrates easily with React, making it ideal for displaying telemetry and event-based data.
