# appsonair-graphql-server-v2/
│
├── .code-review-graph/                    # Code review graph database
│   └── graph.db
│
├── .env                                  # Environment variables
├── .env-sample                           # Environment variables template
├── .eslintrc.json                        # ESLint configuration
├── .editorconfig                         # Editor configuration
├── .gitignore                            # Git ignore rules
├── .gitlab-ci.yml                        # GitLab CI/CD configuration
├── .husky/                               # Git hooks
│   ├── commit-msg
│   ├── pre-commit
│   ├── pre-push
│   └── _/
│       └── husky.sh
│
├── .qodo/                                # Qodo AI configuration
│   ├── agents/
│   └── workflows/
│
├── Dockerfile                            # Docker container configuration
├── README.md                             # Project documentation
├── assets/                               # Static assets
│   └── device.mobileconfig
│
├── package.json                          # Node.js dependencies
├── package-lock.json                     # Locked dependencies
├── sonar-project.properties              # SonarQube configuration
├── bitbucket-pipelines.yml               # Bitbucket CI/CD
│
├── src/                                  # Source code
│   │
│   ├── boot/                             # Boot scripts (data initialization)
│   │   ├── admin-app-member.js
│   │   ├── create-feature.js
│   │   ├── custom-plan/
│   │   ├── data/
│   │   ├── events/
│   │   ├── faq/
│   │   ├── fetch-and-add-app-link-analytics.js
│   │   ├── integration/
│   │   ├── set-app-config-json.js
│   │   ├── set-default-workspace.js
│   │   ├── set-link-value.js
│   │   ├── team-config/
│   │   └── upsert-team-config.js
│   │
│   ├── config/                           # Application configuration
│   │   └── config.js
│   │
│   ├── schema/                           # Database schemas
│   │   ├── api-log/
│   │   ├── app-link-analytics/
│   │   ├── codepush/
│   │   │   ├── migrations/
│   │   │   └── models/
│   │   └── main-server/
│   │       ├── migrations/
│   │       └── models/                   # Sequelize models
│   │
│   ├── services/                         # External service integrations
│   │   └── integrations/
│   │       ├── discord/
│   │       ├── generic/
│   │       ├── google-chat/
│   │       ├── slack/
│   │       └── zapier/
│   │
│   ├── modules/                          # GraphQL modules (feature domains)
│   │   ├── applications/
│   │   ├── app-services/
│   │   ├── code-push/
│   │   ├── common/
│   │   ├── dynamic-link/
│   │   ├── dynamic-link-analytics/
│   │   ├── dynamic-link-config/
│   │   ├── faq/
│   │   ├── feedback/
│   │   ├── group/
│   │   ├── integrations/
│   │   ├── notification/
│   │   ├── policy/
│   │   ├── release/
│   │   ├── release-feedback/
│   │   ├── subscription/
│   │   ├── team/
│   │   └── user/
│   │
│   ├── rest/                             # REST API layer
│   │   ├── middlewares/
│   │   ├── routes/
│   │   ├── modules/
│   │   │   ├── analytics/
│   │   │   ├── app-link/
│   │   │   ├── apps/
│   │   │   ├── code-push/
│   │   │   ├── feedback/
│   │   │   ├── newsletter/
│   │   │   ├── paddle/
│   │   │   ├── plist-details/
│   │   │   ├── signed-url/
│   │   │   ├── stripe/
│   │   │   ├── subscription/
│   │   │   ├── udid-and-app-services/
│   │   │   └── workspace/
│   │   └── functions/
│   │
│   ├── shared-lib/                       # Shared libraries
│   │   ├── storage-v2/                   # Storage abstraction v2
│   │   │   ├── config-v2.js
│   │   │   ├── functions/
│   │   │   └── index.js
│   │   ├── storage/
│   │   ├── paddle/
│   │   ├── queue/                        # Background job queue
│   │   ├── providers/
│   │   │   └── email/
│   │   ├── logger/
│   │   ├── dittofeed/                    # Event tracking
│   │   ├── graphql/
│   │   ├── error-handler/
│   │   ├── constants/
│   │   └── utils/
│   │
│   ├── utils/                            # Utility functions
│   │   ├── auth/
│   │   ├── storage-config/
│   │   ├── code-push/
│   │   ├── cloudflare/
│   │   ├── dittofeed/
│   │   ├── ipa-info/
│   │   ├── intl/
│   │   ├── notifiction-methods/
│   │   ├── rest/
│   │   ├── acumbamail/
│   │   ├── reoon/
│   │   ├── app-link/
│   │   ├── device-mobile-config/
│   │   ├── input-validations/
│   │   └── common.js
│   │
│   ├── scalars/                          # GraphQL scalar types
│   ├── directives/                       # GraphQL directives
│   ├── constants/                        # Application constants
│   ├── pubsub/                           # PubSub system
│   ├── providers/                        # Service providers
│   ├── start-apollo-server.js            # Apollo GraphQL server entry
│   ├── codepush-sequelize-client.js      # CodePush Sequelize client
│   ├── sequelize-client.js               # Main Sequelize client
│   └── logger.js                         # Application logger
│
└── .gitmodules                          # Git submodules configuration