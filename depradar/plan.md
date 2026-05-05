# Depradar\n\n## Plan\n\n## Decisions\n\n## Status\n

Synopsys:
The goal of Depradar is to track dependencies through GitHub as a way for developers to easily monitor dependencies for risk (security risk or abandonment), so tracking things like active commits, velocity, tied in with CVE reports. This would allow developers and teams to quickly identify if dependencies that are in use in their codebase are being abandoned or not being developed in a manner comprehensive with their risk profile protocol, or possibly to maintain audit certification, whatever that may mean. Initially, we create a dashboard of the top 25 repos as kind of a public service. Then users could sign up for a subscription (TBD) that would allow them to link and monitor through GitHub and possibly utilize CI/CD within their codebase to help them manage from an audit perspective or be able to flag security risks or identify deprecated dependencies that are still in use and need to possibly be replaced or updated. 

details:
The MVP should consist of a public service dashboard that identifies/ranks the top 25 dependencies in use, things like chalk, react, etc. How we rank them needs to be determined whether we use Github or NPM or some other - I don't know if GitHub has a ranking system on popularity, or I'm sure there's some sites we could look at that rank them in terms of use. But NPM I think has upwards of 69K dependency packages - can we get the full list? The dashboard should consist of widgets, a widget for each repo with a velocity graph based on the number of commits. It should also have a 30-, 60-, and 90-day historical view toggle. It should also have some little icons denoting the number of CVEs and noting the velocity (whether it's high, medium, or low). Perhaps a risk level, same high, medium, low, that can probably be dialed in. Maybe more of a number between 1 and 100, with color-coding:

- Anything under 15 is green.
- Anything 15 to 30 is yellow.
- 30 to 70 is orange.
- 70 to 100 is red.

Clicking on one of these widgets would load a page for that repo with larger views and links to GitHub for the repo, maybe links to the CVEs that we can pull through the NIST. I think they have an API we need to look at that.


the subscription model needs to be reviewed in more detail but could look something like this:

- Free subscription, one user, one repo, no continuous integration.
- $5/month subscription, one user, up to three repos, no continuous integration.
- $15/month, three users, three repos, (CI/CD Adda ON - $30). 
- $50/month, up to 10 users, 10 repos, (CI/CD Add On - $100) 
- $250/month, up to 25 users, unlimited repos (CI/CD)

With connected users, we will need to connect their GitHub accounts. I don't know what the best way to do that is. I guess if we just use a web hook, or some other method to be determined.  I mean, I definitely see using the webhooks for doing this continuous integration stuff, but then it also seems, technically, it's trivial once you do that to actually do the continuous integration versus having it as an add-on service. I think there's value there, so is there a different way to connect their GitHub account that's not using webhooks? Need to invest in what the easiest  methods are.

need to think of the design on the user dashboard. If we go with a GitHub style where there's a general menu on the left side that shows:

- top repos (same as public dashboard)
- your repos
- alerts
- history

and then a sub menu with the pertinent details. If they've got one repo, we click on that. It's going to show a list of all dependencies within that repo in the standard style widget of the graph with the 30/60/90 views, velocity and risk indicators, CVE information similar to the dashboard. Maybe some other stuff, but I think that kind of gets it going

 -the top repos should show the same public dashboard that's on the main page 
 -Your Repos should show the repos that you've got connected, with the option to add a repo 
 -alerts should allow you a space to select the repos and/or what kind of alerts you'd like to see. You want to see CVE alerts on all of them, and you want to see only alerts if the risk level increases 
 -history would be a history of any dependency repo changes, whether it's CVEs up or down, maybe with a little red or green arrow if it's velocity or risk changes up or down. Possibly some other information, kind of like logging of sorts 

 so when a user connects their GitHub or connects, let's say, their repo, then essentially we want to scan the repo, match that with dependencies, and then give them their dashboard of associated dependencies for that project, as a tree, by project. 
This isn't going to really apply for the larger dependencies that are monitored and managed and developed by the Facebooks, Apples, Googles, and Nvidias of the world, because those are always going to have developers working on them at a probably pretty consistent velocity. It's going to be the smaller ones that are one, two, or ten developers that become really popular because they solve a problem, kind of what we're doing. We're solving a problem, but we need to have a way to add them and monitor them as well in a separate screen 

MVP
For the mmmVP  let's start with the Core Framework and the Public Dashboard. We'll have to figure out how to get or cultivate a list of the top 25. Let's focus on the Widget layout with Velocity as the primary. We'll add the CVE and go ahead and model in the Risk. We'll have to come up with a calculation for how we're going to account for that, but we'll figure out the CVE will be after MVP. We'll do that next, along with some of the other stuff.

Let's focus on the Dashboard, cultivating the initial list and the user setup. We'll just say it's for the free service account, or for the free account for one user, one repo. Make sure we get all the GitHub authentication wired in 


 beyond MVP
 features to be added after MVP deployment 

* We should also have a method for manual monitoring. Let's say you've got a handful of repos in your war chest, but you don't always use them for all the projects. You might want to add those in there just to be able to keep an eye on things.
 * Pull CVE data from nvd.nist.gov, or possibly OWASP. Not sure what the best source is. Probably NIST
 * build out or add functionality for tiered subscription accounts
 * build out continuous integration webhook 
 *  build out an API for users to integrate the data into their own workflows 

