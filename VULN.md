# notes

```bash
OLD=ghcr.io/apache/flink-kubernetes-operator:0315e91
NEW=ghcr.io/sn-harishbajaj/flink-kubernetes-operator:2ff2f66

for IMAGE in "$OLD" "$NEW"; do
  docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    docker-local.artifactory.servicenow.net/gitlab.servicenow.net/snowsk8s/core/nanna/trivy-preloaded-vuln-dbs:0.64.0-vuln-dbs-20250902 \
    image \
      --scanners vuln \
      --skip-db-update \
      --skip-java-db-update \
      --format json \
      --quiet \
      --cache-dir /home/scanner/.cache/trivy \
      "$IMAGE" \
  | jq -r '
      .Results[]? |
      select(.Vulnerabilities) |
      .Class as $class |
      .Vulnerabilities[]? |
      "\(.VulnerabilityID),\($class),\(.PkgID),\(.PkgName),\(.PrimaryURL),\(.CVSS.nvd.V3Score)"
    ' \
  | grep -E "CVE-2017-11164|CVE-2022-41409|CVE-2016-20013|CVE-2022-4899" \
  | sort -k1,1 -t',' \
  | { echo "VULN_ID,CLASS,PKG_ID,PKG_NAME,URL,CVSS"; cat; } \
  | column -t -s ',' \
  | bat -l csv --pager=never
done

WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ STDIN
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ VULN_ID         CLASS    PKG_ID                             PKG_NAME      URL                                         CVSS
   2   │ CVE-2016-20013  os-pkgs  libc-bin@2.35-0ubuntu3.10          libc-bin      https://avd.aquasec.com/nvd/cve-2016-20013  7.5
   3   │ CVE-2016-20013  os-pkgs  libc6@2.35-0ubuntu3.10             libc6         https://avd.aquasec.com/nvd/cve-2016-20013  7.5
   4   │ CVE-2016-20013  os-pkgs  locales@2.35-0ubuntu3.10           locales       https://avd.aquasec.com/nvd/cve-2016-20013  7.5
   5   │ CVE-2017-11164  os-pkgs  libpcre3@2:8.39-13ubuntu0.22.04.1  libpcre3      https://avd.aquasec.com/nvd/cve-2017-11164  7.5
   6   │ CVE-2022-41409  os-pkgs  libpcre2-8-0@10.39-3ubuntu0.1      libpcre2-8-0  https://avd.aquasec.com/nvd/cve-2022-41409  7.5
   7   │ CVE-2022-4899   os-pkgs  libzstd1@1.4.8+dfsg-3build1        libzstd1      https://avd.aquasec.com/nvd/cve-2022-4899   7.5
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ STDIN
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ VULN_ID         CLASS    PKG_ID                    PKG_NAME  URL                                         CVSS
   2   │ CVE-2016-20013  os-pkgs  libc-bin@2.39-0ubuntu8.5  libc-bin  https://avd.aquasec.com/nvd/cve-2016-20013  7.5
   3   │ CVE-2016-20013  os-pkgs  libc6@2.39-0ubuntu8.5     libc6     https://avd.aquasec.com/nvd/cve-2016-20013  7.5
   4   │ CVE-2016-20013  os-pkgs  locales@2.39-0ubuntu8.5   locales   https://avd.aquasec.com/nvd/cve-2016-20013  7.5
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```
