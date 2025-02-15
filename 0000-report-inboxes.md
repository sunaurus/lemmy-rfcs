- Feature Name: report-inboxes
- Start Date: 2024-02-20
- RFC PR: [LemmyNet/rfcs#0000](https://github.com/LemmyNet/rfcs/pull/0000)
- Lemmy Issue: [LemmyNet/lemmy#0000](https://github.com/LemmyNet/lemmy/issues/0000)

# Summary

Rather than combining all reports into a single report inbox, we should allow users to select whether they are reporting to mods or admins, and we should split reports into different inboxes based on that selection.

# Motivation

The current approach has some shortcomings:

* Users are not currently able to bypass mods and report directly to admins - this may allow mods to conceal instance rule breaking in specific communities
* Admins are not aware of community rules, so they may wish to take no action for most community rule breaking reports. However, if an admin resolves such a report, the relevant community mods most likely never see it.
* Different instances may have different rules, but somebody resolving a report on one instance will resolve it for other instances as well, thus potentially resulting in missed reports.
* Mods might take local action on a report and mark it as resolved even in cases where a user should be banned from the entire instance. In this case, admins are very unlikely to see the report.

# Guide-level explanation

### When creating reports, users will be able to select if it's a mod report, or an admin report (or both)

![image](https://github.com/sunaurus/lemmy-rfcs/assets/5356547/9a21b527-6c88-4024-b287-3371d77688f4)

**Note: labels on the sreenshot are illustrative, actual labels can be more user-friendy.** Maybe something like:
* Breaks community rules (report sent to moderators)
* Breaks instance rules (report sent to admins)

### Instead of the current single report inbox, there will be three different kinds of inboxes

* Admin reports - show all reports sent to admins (only visible to admins) 
* Mod reports - show all reports sent to mods for any communities the user moderates (visible to admins in case they are explicit mods in any communities)
   * This is equivalent to the report view that mods currently have in Lemmy already
* All reports - Shows a view of all (admin and mod) reports, only visible to admins
   * This is akin to the current 0.19.3 admin report view, and would allow admins to still keep an eye on mod actions on their instance if they wish
    
The UI wouldn't need to change for mods, but for admins, there would be a new selection at the top of the reports page (the "mod reports" tab would only be visible if the admin is also a mod in any community):
![image](https://github.com/sunaurus/lemmy-rfcs/assets/5356547/cc4ad68c-6e85-4cd9-b324-131c06951cb3)

### Resolving reports should be more granular

* Reports in the "admin reports" tab can only be manually resolved for admins of the local instance
   * To reduce overhead, **banning the reported user on the user's home instance + removing reported content should automatically resolve reports for remote admins as well**.
* Reports in the "mod reports" tab should be manually resolved by relevant mods (including admins, if they are explicit mods in the relevant community).
   * To reduce overhead, **admins banning the reported user on the community instance OR the user's home instance + removing reported content should automatically resolve reports for mods as well**
* Admins could still resolve reports in the "all reports" tab
   * If it's not an admin report, and not a mod report from a community the admin explicitly moderates, then there should be an additional warning/confirmation when resolving a report here. This is to prevent cases of admins accidentally preventing mods from moderating according to their own community rules.

To further clarify automatic resolution of reports: in any case where there is no further action possible, the report should be automatically resolved.
  
### Mods should be able to escalate reports to admins

This would generate a corresponding report in the admin inbox.

# Reference-level explanation

* In the UI, changes are needed for both reporting as well as the reports inbox views
* In the database and API, we should split reports by intended audience
* Federation needs to be changed as well in order to allow distinguishing the report target audience

# Drawbacks

It might make reporting slightly more confusing for end users - the mod/admin distinction might not be fully clear to all.

# Rationale and alternatives

Alternatively, we could make reporting **even more** granular. It would be possible to allow users to select only a specific instances admins as the intended report audience, for example.
However, I think this has several downsides:
* Makes the report UI even more confusing
* Potentially takes away valuable information from other admins (imagine a user only reports CSAM to their own instances admins, while leaving the offending post authors home admins in the dark)

# Prior art

Most other social networks allow users to select whether they are reporting a violation of community rules, or site rules as whole.

# Unresolved questions

Does ActivityPub properly support splitting up reports like this?

# Future possibilities

In the future, it might be a nice addition to have some automation to always escalate to admins, even if they're submitted as mod reports, based on report keywords. For example, "CSAM", "Spam", etc.
