# satellite-content-view

This project consists of two roles: 

satellite-content-view and content_view_version_cleanup

The site.yml contains all the variables

satellite_deployment_organization: 'QBE'

satellite_deployment_admin_username: 'admin'

satellite_deployment_admin_password: redhat123

satellite_lifecycle: [Library, canary]

satellite_content_view: cv-rhel8

satellite_deployment_server: "https://rhsatellite.example.net" 

satellite_content_view_version_cleanup_keep: 0


To execute publish/promote: use --tags satellite-content-view

To execute cleanup: use --tags satellite-content-cleanup
