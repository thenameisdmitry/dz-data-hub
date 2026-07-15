# Request access to a dataset

<tldr>

**Audience:** anyone at DZ Software who needs warehouse data

**Time to complete:** 5 minutes to request; approval usually within one business day

**Prerequisites:** a Snowflake account (provisioned automatically for all employees)

</tldr>

Access to warehouse datasets is granted through domain-scoped reader
roles — for example, `DATA_READER_OPS` for processing and operations
data. Each role is owned by the team that owns the data, and every
grant is reviewed and auditable: we handle client financial data, so
access follows the least-privilege principle by default.

## Before you start

Check whether you already have the role you need: in Snowflake, run
`SHOW GRANTS TO USER <your_username>;` — or simply try querying the
dataset. If you can query it, you already have access.

To find out which role a dataset requires, open its catalog page and
check the **Access** section — for example,
[fct_transactions](fct-transactions.md#access) requires
`DATA_READER_OPS`.

<procedure title="Request a reader role" id="request-reader-role">
    <step>
        Open the <ui-path>DATA</ui-path> project in YouTrack and click
        <control>New Issue</control>.
    </step>
    <step>
        Set the issue type to <control>Access Request</control>.
    </step>
    <step>
        <p>Fill in the request form:</p>
        <list>
            <li><control>Role</control> — the reader role from the dataset's catalog page.</li>
            <li><control>Business justification</control> — one or two sentences on what you need the data for. "Ad-hoc analysis" is a valid justification; an empty field is not.</li>
            <li><control>Duration</control> — <code>permanent</code> for ongoing work or <code>90 days</code> for one-off projects.</li>
        </list>
    </step>
    <step>
        Submit the issue. It is automatically routed to the data owner
        of the requested domain for approval.
    </step>
    <step>
        After approval, the grant is applied by the next
        infrastructure sync, which runs every 4 hours. You will get a
        YouTrack notification when the role is active.
    </step>
    <step>
        Verify access by running a simple query against the dataset,
        for example <code>SELECT COUNT(*) FROM
        marts.fct_transactions;</code>
    </step>
</procedure>

> Access requests are approved by data owners, not by the Data
> Platform team. If a request is stuck for more than one business
> day, ping the owner listed on the dataset's catalog page — not
> the #data-platform channel.
> {style="tip"}

## Troubleshooting

**The query fails with "Object does not exist or not authorized" after
approval.** The infrastructure sync may not have run yet — it applies
grants every 4 hours. If more than 4 hours have passed, comment on
your YouTrack issue.

**You need access urgently.** Mark the issue with the
<control>Urgent</control> tag and post a link to it in the
`#data-access` channel. Owners monitor the channel during business
hours (CET).

**You need write access.** Warehouse tables are never written to
directly by individuals — all changes go through the Data Platform
team's managed pipelines. Open a <control>Change Request</control>
issue in the <ui-path>DATA</ui-path> project instead.