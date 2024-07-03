# Linkedin API for Python

Programmatically search profiles, send messages, and find jobs. All with a regular Linkedin user account.

No "official" API access required - just use a valid Linkedin account!

**Caution**: This library is not officially supported by LinkedIn. Using it might violate LinkedIn's Terms of Service. Use it at your own risk.

## Installation

> Python >= 3.10 required

### Quick Start
> Script Client
```
linkedin = LinkedInScriptApi(credentials["username"], credentials["password"])
jobs = linkedin.search_jobs("Software", total_jobs = 10_000)
```

>Async Version

```
session = AsyncClient()
client = AsyncLinkedInClient(session=session)
linkedin = AsyncLinkedIn(client)
await linkedin.authenticate(credentials["username"], credentials["password"])
await linkedin.get_profile_privacy_settings("khalid-a-53a190142")
profile = await linkedin.search_people(current_company=[CompanyID.GOOGLE], past_companies=[CompanyID.APPLE], include_private_profiles=True)
company = await linkedin.get_company_updates(public_id="google")
await linkedin.get_organization("google")
jobs = await linkedin.search_jobs(
    "Software Engineer",
    sort_by=SortBy.DATE,
    location=GeoID.USA,
    remote=[LocationType.ONSITE],
    limit=10,
)
if jobs:
    for job in jobs.elements:
        job_complete = await linkedin.get_job(job.tracking_urn.split(":")[-1])
        job_skills = await linkedin.get_job_skills(job.tracking_urn.split(":")[-1])
    print(job_complete)
await linkedin.search({"keywords": "software"})
res = await linkedin.search_people(keywords="software",include_private_profiles=True)
await linkedin._close()
```

> Sync Version
```
session = Client()
client = LinkedInClient(session=session)
linkedin = LinkedIn(client)
linkedin.authenticate(credentials["username"], credentials["password"])

linkedin.get_profile_privacy_settings("khalid-a-53a190142")
profile = linkedin.search_people(current_company=[CompanyID.GOOGLE], past_companies=[CompanyID.APPLE], include_private_profiles=True)
company = linkedin.get_company_updates(public_id="google")
linkedin.get_organization("google")
jobs = linkedin.search_jobs(
    "Software Engineer",
    sort_by=SortBy.DATE,
    location=GeoID.USA,
    remote=[LocationType.ONSITE],
    limit=10,
)
if jobs:
    for job in jobs.elements:
        job_complete = linkedin.get_job(job.tracking_urn.split(":")[-1])
        job_skills = linkedin.get_job_skills(job.tracking_urn.split(":")[-1])
    print(job_complete)
linkedin.search({"keywords": "software"})
res = linkedin.search_people(keywords="software",include_private_profiles=True)
linkedin._close()
session = Client()
client = LinkedInClient(session=session)
linkedin = LinkedIn(client)
linkedin.authenticate(credentials["username"], credentials["password"])

linkedin.get_profile_privacy_settings("khalid-a-53a190142")
profile = linkedin.search_people(current_company=[CompanyID.GOOGLE], past_companies=[CompanyID.APPLE], include_private_profiles=True)
company = linkedin.get_company_updates(public_id="google")
linkedin.get_organization("google")
jobs = linkedin.search_jobs(
    "Software Engineer",
    sort_by=SortBy.DATE,
    location=GeoID.USA,
    remote=[LocationType.ONSITE],
    limit=10,
)
if jobs:
    for job in jobs.elements:
        job_complete = linkedin.get_job(job.tracking_urn.split(":")[-1])
        job_skills = linkedin.get_job_skills(job.tracking_urn.split(":")[-1])
    print(job_complete)
linkedin.search({"keywords": "software"})
res = linkedin.search_people(keywords="software",include_private_profiles=True)
linkedin._close()        session = Client()
client = LinkedInClient(session=session)
linkedin = LinkedIn(client)
linkedin.authenticate(credentials["username"], credentials["password"])

linkedin.get_profile_privacy_settings("khalid-a-53a190142")
profile = linkedin.search_people(current_company=[CompanyID.GOOGLE], past_companies=[CompanyID.APPLE], include_private_profiles=True)
company = linkedin.get_company_updates(public_id="google")
linkedin.get_organization("google")
jobs = linkedin.search_jobs(
    "Software Engineer",
    sort_by=SortBy.DATE,
    location=GeoID.USA,
    remote=[LocationType.ONSITE],
    limit=10,
)
if jobs:
    for job in jobs.elements:
        job_complete = linkedin.get_job(job.tracking_urn.split(":")[-1])
        job_skills = linkedin.get_job_skills(job.tracking_urn.split(":")[-1])
    print(job_complete)
linkedin.search({"keywords": "software"})
res = linkedin.search_people(keywords="software",include_private_profiles=True)
linkedin._close()
```

## Documentation



## Disclaimer

This library is not endorsed or supported by LinkedIn. It is an unofficial library intended for educational purposes and personal use only. By using this library, you agree to not hold the author or contributors responsible for any consequences resulting from its usage.

## Contributing

We welcome contributions! [Learn how to find endpoints](#find-new-endpoint).

## Development

### Development installation

TODO

### Troubleshooting

#### I keep getting a `CHALLENGE`

Linkedin will throw you a curve ball in the form of a Challenge URL. We currently don't handle this, and so you're kinda screwed. We think it could be only IP-based (i.e. logging in from different location). Your best chance at resolution is to log out and log back in on your browser.

**Known reasons for Challenge** include:

- 2FA
- Rate-limit - "It looks like you’re visiting a very high number of pages on LinkedIn.". Note - n=1 experiment where this page was hit after ~900 contiguous requests in a single session (within the hour) (these included random delays between each request), as well as a bunch of testing, so who knows the actual limit.

Please add more as you come across them.

#### Search problems

- Mileage may vary when searching general keywords like "software" using the standard `search` method. They've recently added some smarts around search whereby they group results by people, company, jobs etc. if the query is general enough. Try to use an entity-specific search method (i.e. search_people) where possible.

## How it works

This project attempts to provide a simple Python interface for the Linkedin API.

> Do you mean the [legit Linkedin API](https://developer.linkedin.com/)?

NO! To retrieve structured data, the [Linkedin Website](https://linkedin.com) uses a service they call **Voyager**. Voyager endpoints give us access to pretty much everything we could want from Linkedin: profiles, companies, connections, messages, etc. - anything that you can see on linkedin.com, we can get from Voyager.

This project aims to provide complete coverage for Voyager.

[How does it work?](#deep-dive)

### Deep dive

Voyager endpoints look like this:

```text
https://www.linkedin.com/voyager/api/identity/profileView/tom-quirk
```

Or, more clearly

```text
 ___________________________________ _______________________________
|             base path             |            resource           |
https://www.linkedin.com/voyager/api /identity/profileView/tom-quirk
```

They are authenticated with a simple cookie, which we send with every request, along with a bunch of headers.

To get a cookie, we POST a given username and password (of a valid Linkedin user account) to `https://www.linkedin.com/uas/authenticate`.

### Find new endpoints

We're looking at the Linkedin website and we spot some data we want. What now?

The following describes the most reliable method to find relevant endpoints:

1. `view source`
1. `command-f`/search the page for some keyword in the data. This will exist inside of a `<code>` tag.
1. Scroll down to the **next adjacent element** which will be another `<code>` tag, probably with an `id` that looks something like

   ```html
   <code style="display: none" id="datalet-bpr-guid-3900675">
     {"request":"/voyager/api/identity/profiles/tom-quirk/profileView","status":200,"body":"bpr-guid-3900675"}
   </code>
   ```

The value of `request` is the url! 🤘

You can also use the `network` tab in you browsers developer tools, but you will encounter mixed results.

### How Clients query Voyager

linkedin.com uses the [Rest-li Protocol](https://linkedin.github.io/rest.li/spec/protocol) for querying data. Rest-li is an internal query language/syntax where clients (like linkedin.com) specify what data they want. It's conceptually similar to the GraphQL.

Here's an example of making a request for an organisation's `name` and `groups` (the Linkedin groups it manages):

```text
/voyager/api/organization/companies?decoration=(name,groups*~(entityUrn,largeLogo,groupName,memberCount,websiteUrl,url))&q=universalName&universalName=linkedin
```

The "querying" happens in the `decoration` parameter, which looks like the following:

```text
(
    name,
    groups*~(entityUrn,largeLogo,groupName,memberCount,websiteUrl,url)
)
```

Here, we request an organisation name and a list of groups, where for each group we want `largeLogo`, `groupName`, and so on.

Different endpoints use different parameters (and perhaps even different syntaxes) to specify these queries. Notice that the above query had a parameter `q` whose value was `universalName`; the query was then specified with the `decoration` parameter.

In contrast, the `/search/cluster` endpoint uses `q=guided`, and specifies its query with the `guided` parameter, whose value is something like

```text
List(v->PEOPLE)
```

It could be possible to document (and implement a nice interface for) this query language - as we add more endpoints to this project, I'm sure it will become more clear if such a thing would be possible (and if it's worth it).

### Release a new version
