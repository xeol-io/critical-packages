# NuGet

```sql
DECLARE
  Sys STRING DEFAULT 'NUGET';

WITH
  HighestReleases AS (
    SELECT
      Name,
      Version,
    FROM (
      SELECT
        Name,
        Version,
        ROW_NUMBER()
          OVER (PARTITION BY
                  Name
                ORDER BY
                  VersionInfo.Ordinal DESC) AS RowNumber
        FROM
          `bigquery-public-data.deps_dev_v1.PackageVersionsLatest`
        WHERE
          System = Sys
          AND VersionInfo.IsRelease)
    WHERE RowNumber = 1),
  DependencyCounts AS (
    SELECT
      DependencyGroupsDependencies.Name AS DependencyName,
      COUNT(*) AS DependencyCount
    FROM
      `bigquery-public-data.deps_dev_v1.NuGetRequirementsLatest`,
      UNNEST(DependencyGroups) AS DependencyGroups,
      UNNEST(DependencyGroups.Dependencies) AS DependencyGroupsDependencies
    GROUP BY
      DependencyName)

SELECT
  DC.DependencyName,
  HR.Version AS LatestVersion,
  DC.DependencyCount
FROM
  DependencyCounts DC
JOIN
  HighestReleases HR
ON
  DC.DependencyName = HR.Name
ORDER BY
  DependencyCount DESC
LIMIT
  5000;
```

# All Others

```sql
DECLARE
  Sys STRING DEFAULT 'GO';

WITH
  HighestReleases AS (
    SELECT
      Name,
      Version,
    FROM (
      SELECT
        Name,
        Version,
        ROW_NUMBER()
          OVER (PARTITION BY
                  Name
                ORDER BY
                  VersionInfo.Ordinal DESC) AS RowNumber
        FROM
          `bigquery-public-data.deps_dev_v1.PackageVersionsLatest`
        WHERE
          System = Sys
          AND VersionInfo.IsRelease)
    WHERE RowNumber = 1)

SELECT
  D.Dependency.Name,
  D.Dependency.Version,
  COUNT(*) AS NDependents
FROM
  `bigquery-public-data.deps_dev_v1.DependenciesLatest` AS D
JOIN
  HighestReleases AS H
USING
  (Name, Version)
WHERE
  D.System = Sys
GROUP BY
  D.Dependency.Name,
  D.Dependency.Version
ORDER BY
  NDependents DESC
LIMIT
  5000;
```