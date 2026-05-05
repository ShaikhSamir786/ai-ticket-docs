# appsonair-codepush-server-ts
│
├── docker/
│   ├── docker-compose.dev.yml
│   ├── .env.docker
│   └── startup.sh
│
├── docker-compose.dev.yml
├── Dockerfile
├── .editorconfig
├── .env
├── .env.sample
├── eslint.config.mjs
├── .eslintrc.json
├── nodemon.json
├── package.json
├── package-lock.json
├── README.md
├── sonar-project.properties
│
├── .husky/                                  # Git hooks
│   ├── commit-msg
│   ├── pre-commit
│   └── _/                                    # Husky internal files
│       ├── applypatch-msg
│       ├── commit-msg
│       ├── husky.sh
│       ├── post-applypatch
│       ├── post-checkout
│       ├── post-merge
│       ├── post-rewrite
│       ├── pre-applypatch
│       ├── pre-auto-gc
│       ├── pre-commit
│       ├── pre-merge-commit
│       ├── prepare-commit-msg
│       ├── pre-push
│       └── pre-rebase
│
└── src/
    │
    ├── index.ts                              # Application entry point
    ├── logger.ts                             # Logger setup
    ├── config.ts                             # App configuration
    ├── api-log-sequelize-client.ts           # API log DB client
    ├── sequelize-client.ts                   # Main DB client
    ├── sequelize-client-main-server.ts       # Main server DB client
    ├── sequelize-client-app-link-analytics-server.ts  # Analytics DB client
    ├── redis-client.ts                       # Redis client (main)
    ├── redis-client-app-link.ts              # Redis client (app-link)
    ├── sentry.ts                             # Sentry error tracking
    │
    ├── boot/                                 # Application bootstrapping
    │   ├── index.ts
    │   ├── boot-logger.ts
    │   └── keep-alive.ts
    │
    ├── logger/                               # Logger utilities
    │   └── logger.ts
    │
    ├── providers/                            # External service providers
    │   ├── index.ts
    │   └── email.ts
    │
    ├── constants/                            # Application constants
    │   ├── api-constants.ts
    │   ├── error-type.ts
    │   ├── language-constants.ts
    │   └── redis-constant.ts
    │
    ├── enums/                                # TypeScript enumerations
    │   ├── api.ts
    │   ├── error-type.ts
    │   ├── language.ts
    │   └── service.ts
    │
    ├── types/                                # TypeScript type definitions
    │   ├── common.d.ts
    │   └── constant.d.ts
    │
    ├── functions/                            # Shared utility functions
    │   └── common.ts
    │
    ├── utils/                                # Utility modules
    │   ├── auth/
    │   │   ├── add-request-meta-to-ctx.ts
    │   │   ├── encryption.ts
    │   │   ├── generate-token.ts
    │   │   ├── get-decoded-token.ts
    │   │   └── jwt/
    │   │       ├── generate-access-token.ts
    │   │       ├── generate-refresh-token.ts
    │   │       ├── generate-reset-token.ts
    │   │       ├── generate-verification-token.ts
    │   │       ├── index.ts
    │   │       └── decode-token.ts
    │   │
    │   ├── converter.ts
    │   ├── cors-options.ts
    │   ├── dittofeed/
    │   │   ├── common/
    │   │   │   └── init-dittofeed.ts
    │   │   ├── functions/
    │   │   │   ├── fetch-all-team-members.ts
    │   │   │   └── get-owner-details.ts
    │   │   ├── usage-alerts/
    │   │   │   └── codepush/
    │   │   │       └── check-codepush-download-quota.ts
    │   │   └── shared/
    │   │       └── send-threshold-alert.ts
    │   ├── file-upload-manager.ts
    │   ├── generate-random-key.ts
    │   ├── generate-secure-key.ts
    │   ├── intl/
    │   │   ├── add-locale-service-to-ctx.ts
    │   │   ├── i18n-config.ts
    │   │   ├── locale-service.ts
    │   │   └── locales/
    │   │       ├── en.json
    │   │       └── es.json
    │   ├── messages.ts
    │   ├── query-length-middleware.ts
    │   ├── redis-utils.ts
    │   ├── rest/
    │   │   ├── api-error.ts
    │   │   ├── decode-token.ts
    │   │   ├── generate-response.ts
    │   │   └── get-user.ts
    │   └── common.ts
    │
    ├── rest/                                 # REST API modules
    │   ├── middlewares/
    │   │   ├── error-handler.ts
    │   │   ├── handle-url-middleware.ts
    │   │   ├── is-authenticated.ts
    │   │   ├── is-user-in-team.ts
    │   │   ├── rate-limit.ts
    │   │   ├── restrict-download-middleware.ts
    │   │   ├── set-locale-service-in-req.ts
    │   │   ├── set-log-info-in-req.ts
    │   │   └── workspace-header.ts
    │   │
    │   ├── routes/
    │   │   ├── index.ts
    │   │   └── v1.ts
    │   │
    │   ├── modules/
    │   │   │
    │   │   ├── access-key/                  # Access key management
    │   │   │   ├── access-key.d.ts
    │   │   │   ├── access-key-logger.ts
    │   │   │   └── v1/
    │   │   │       ├── controllers/
    │   │   │       │   ├── create-access-key.ts
    │   │   │       │   ├── delete-access-key.ts
    │   │   │       │   ├── get-access-key.ts
    │   │   │       │   └── get-access-keys.ts
    │   │   │       ├── routes.ts
    │   │   │       └── services/
    │   │   │           ├── create-access-key.ts
    │   │   │           ├── delete-access-key.ts
    │   │   │           ├── get-access-key.ts
    │   │   │           ├── get-access-keys.ts
    │   │   │           └── index.ts
    │   │   │
    │   │   ├── account/                      # Account management
    │   │   │   ├── account.d.ts
    │   │   │   ├── account-logger.ts
    │   │   │   └── v1/
    │   │   │       ├── controllers/
    │   │   │       │   ├── create-account.ts
    │   │   │       │   ├── delete-account.ts
    │   │   │       │   ├── get-account.ts
    │   │   │       │   └── update-account.ts
    │   │   │       ├── routes.ts
    │   │   │       └── services/
    │   │   │           ├── create-account.ts
    │   │   │           ├── delete-account.ts
    │   │   │           ├── get-account.ts
    │   │   │           ├── index.ts
    │   │   │           └── update-account.ts
    │   │   │
    │   │   ├── app/                          # App management
    │   │   │   ├── app-logger.ts
    │   │   │   ├── appTypes.d.ts
    │   │   │   └── v1/
    │   │   │       ├── controllers/
    │   │   │       │   ├── app.ts
    │   │   │       │   ├── apps.ts
    │   │   │       │   ├── collaborators.ts
    │   │   │       │   ├── create-app.ts
    │   │   │       │   └── delete-app.ts
    │   │   │       ├── routes.ts
    │   │   │       └── services/
    │   │   │           ├── app.ts
    │   │   │           ├── apps.ts
    │   │   │           ├── create-app.ts
    │   │   │           ├── delete-app.ts
    │   │   │           └── index.ts
    │   │   │
    │   │   ├── app-deployment/               # Deployment management
    │   │   │   ├── deployment-logger.ts
    │   │   │   ├── deploymentTypes.d.ts
    │   │   │   └── v1/
    │   │   │       ├── controllers/
    │   │   │       │   ├── add-app-deployment.ts
    │   │   │       │   ├── app-deployments.ts
    │   │   │       │   ├── app-deployment.ts
    │   │   │       │   ├── create-deployment-release.ts
    │   │   │       │   ├── delete-app-deployment.ts
    │   │   │       │   ├── get-deployment-history.ts
    │   │   │       │   ├── get-deployment-metrics.ts
    │   │   │       │   ├── rollback-deployment.ts
    │   │   │       │   ├── update-app-deployment.ts
    │   │   │       │   └── update-deployment-release.ts
    │   │   │       ├── functions/
    │   │   │       │   ├── create-metrics-response.ts
    │   │   │       │   ├── get-last-package-hash-with-same-app-version.ts
    │   │   │       │   └── package-manifest.ts
    │   │   │       ├── routes.ts
    │   │   │       └── services/
    │   │   │           ├── add-app-deployment.ts
    │   │   │           ├── app-deployments.ts
    │   │   │           ├── app-deployment.ts
    │   │   │           ├── create-deployment-release.ts
    │   │   │           ├── delete-app-deployment.ts
    │   │   │           ├── get-deployment-history.ts
    │   │   │           ├── get-deployment-metrics.ts
    │   │   │           ├── index.ts
    │   │   │           ├── rollback-deployment.ts
    │   │   │           ├── update-app-deployment.ts
    │   │   │           ├── update-check.ts
    │   │   │           └── update-deployment-release.ts
    │   │   │
    │   │   ├── auth/                         # Authentication
    │   │   │   ├── auth-logger.ts
    │   │   │   └── v1/
    │   │   │       └── routes.ts
    │   │   │
    │   │   ├── blob/                         # Blob/file storage
    │   │   │   ├── blob.d.ts
    │   │   │   ├── blob-logger.ts
    │   │   │   └── v1/
    │   │   │       ├── routes.ts
    │   │   │       └── services/
    │   │   │           ├── create-blob.ts
    │   │   │   ├── get-blob.ts
    │   │   │   ├── index.ts
    │   │   │   └── update-blob.ts
    │   │   │
    │   │   ├── common/                       # Shared utilities
    │   │   │   ├── common-logger.ts
    │   │   │   └── functions/
    │   │   │       └── apps.ts
    │   │   │
    │   │   └── reports/                      # Reporting
    │   │       ├── reports-logger.ts
    │   │   │   ├── reportTypes.d.ts
    │   │       └── v1/
    │   │           ├── controllers/
    │   │           │   ├── report-status-deploy.ts
    │   │           │   └── report-status-download.ts
    │   │           ├── functions/
    │   │           │   └── report-status.ts
    │   │           ├── routes.ts
    │   │           └── services/
    │   │               ├── index.ts
    │   │               ├── report-status-deploy.ts
    │   │               └── report-status-download.ts
    │   │
    │   └── types/
    │       └── request.d.ts
    │
    ├── schema/                               # Database schemas (Sequelize)
    │   │
    │   ├── api-log/                          # API logging schema
    │   │   ├── migrations/
    │   │   │   └── config.js
    │   │   ├── models/
    │   │   │   └── api-log.model.js
    │   │   └── .sequelizerc
    │   │
    │   ├── app-link-analytics/               # Dynamic link analytics schema
    │   │   ├── migrations/
    │   │   │   └── config.js
    │   │   ├── models/
    │   │   │   ├── app-link-analytics-cron.js
    │   │   │   ├── app-link-analytics-day.model.js
    │   │   │   ├── dynamic-link-analytics.model.js
    │   │   │   └── dynamic-link-referral.model.js
    │   │   └── .sequelizerc
    │   │
    │   ├── codepush/                         # CodePush schema
    │   │   ├── migrations/
    │   │   │   ├── config.js
    │   │   │   └── scripts/
    │   │   │       ├── 20250306044203-add-app-version-in-bundle-download.js
    │   │   │       ├── 20250416102208-add-columns-in-metrics-model.js
    │   │   │       ├── 20250605130144-add-type-in-access-key.js
    │   │   │       ├── 20250617093815-add-ref-id-in-deployment.js
    │   │   │       └── 20250619051514-add-column-blob-provider-in-packages-model.js
    │   │   ├── models/
    │   │   │   ├── access-key.model.js
    │   │   │   ├── account.model.js
    │   │   │   ├── app.model.js
    │   │   │   ├── blob.model.js
    │   │   │   ├── bundle-download.model.js
    │   │   │   ├── codepush-analytics-cloudflare-chunk-log.js
    │   │   │   ├── cron-log.js
    │   │   │   ├── deployment.model.js
    │   │   │   ├── history.model.js
    │   │   │   ├── metric.model.js
    │   │   │   ├── package.model.js
    │   │   │   └── .sequelizerc
    │   │   │
    │   └── main-server/                      # Main server schema
    │       ├── migrations/
    │       │   ├── config.js
    │       │   └── local/
    │       │       ├── 20220325090949-modify_app-share_add_new_field_link.js
    │       │       ├── 20220405104306-modify_application_detail_add_new_fields.js
    │       │       ├── 20220405115817-modify_application_remove_fields.js
    │       │       ├── 20220410194531-modify_application_rename_fields.js
    │       │       ├── 20220502092706-modify_team_remove_logo_value.js
    │       │       ├── 20220602095218-made_appLogo_default_value_to_null.js
    │       │       ├── 20220603044335-make_allowNull_true_for_appLogo_column.js
    │       │       ├── 20220608114917-added_new_filed_linkExpiration_in_applicationShare_model.js
    │       │       ├── 20220628054906-added_refreshToken_field_in_user.js
    │       │       ├── 20220708092231-add-application_detail_id-in-app_share-model.js
    │       │       ├── 20220708093300-change-type-of-sub_link_id-to-string-in-app_share-model.js
    │       │       ├── 20220711060904-update-link-in-application_share-model.js
    │       │       ├── 220220901072037-add-platform-field-in-notification.js
    │       │       ├── 20221124074655-update-release_note-string-length.js
    │       │       └── 20230510060053-add-tags-field-in-application-model.js
    │       └── .sequelizerc
    │
    └── shared-lib/                           # Shared libraries (git submodules)
        │
        ├── constants/
        ├── dittofeed/
        ├── error-handler/
        ├── graphql/
        │   ├── plugins/
        │   └── validation-rules/
        ├── logger/
        ├── paddle/
        ├── providers/
        ├── queue/                            # Queue workers
        │   ├── add-api-log/
        │   ├── add-app-collaborator/
        │   ├── add-app-link-analytics/
        │   ├── add-app-link-handler/
        │   ├── add-clickhouse-codepush-analytics/
        │   ├── add-dynamic-link-analytics/
        │   ├── add-dynamic-link-import-csv/
        │   ├── add-purge-req-worker/
        │   ├── cancel-addon-subscription/
        │   ├── downgrade-subscription-plan/
        │   ├── fetch-and-store-app-link-analytics/
        │   ├── queue.logger.js
        │   ├── queue-constant.js
        │   ├── remove-app-collaborator/
        │   ├── server-force-deployment-queue/
        │   ├── store-codepush-bundle-download/
        │   ├── store-codepush-history/
        │   ├── store-codepush-metrics/
        │   └── store-codepush-metrics-v2/
        ├── reoon/
        ├── storage/
        ├── storage-v2/
        ├── stripe/
        └── utils/