# Add Helm Chart Repositories to Rancher

When we installed Longhorn, we used the default Helm Chart repositories installed with Rancher.  Now we will add our own private repostitory to install our own application.

## Step 1 - Log in to Rancher

## Step 2 - Add Harbor to Rancher's Repositories

1. Select your cluster

2. On the left side of the page, Select `Apps`

3. Under `Apps` select `Repositories`

This will show all of the Helm Chart repositories available to install applications

4. Select `Create`

5. Name it `harbor` (all lowercase)

6. In the `url` block, add `https://harbor.vip.ark1.soroc.mil/chartrepo/stud3`

7. Select `Create`

Note that there is an error.  This is because we used self-signed certs to stand up harbor

8. To the right of the harbor repo select the 3 dots

9. Select `Edit YAML`

10. Add the line `insecureSkipTLSVerify: true` under `spec`

```yaml
apiVersion: catalog.cattle.io/v1
kind: ClusterRepo
metadata:
  name: harbor
spec:
  insecureSkipTLSVerify: true
  url: https://harbor.vip.ark1.soroc.mil/chartrepo/instantconnect
```

11. Select `Save`
Harbor should now be displayed green

12. On the left side of the page, select `Charts`

13. In the search bar, search for your helm-chart and view it

## Step 3 - Complete Lab

### Close Lab issue in Gitlab

- Open your student project in GitLab.

- Find the issue that covers this lab.

- Add comments to the issue indicating that you have finished the lab.

- Close the issue.
```