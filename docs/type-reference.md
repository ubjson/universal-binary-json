# Type Reference

The table below is a quick-reference for folks working closely with the Universal Binary JSON format that want all the information at their finger tips:

<table id="type-ref-table" style="width: 100%;" border="0">
<thead>
<tr>
<td>Type</td>
<td>Size</td>
<td>Marker</td>
<td>Length</td>
<td>Data Payload</td>
</tr>
</thead>
<tbody>
<tr>
<td colspan="5">

> Value Types

</td>
</tr>
<tr>
<td><a href="value-types#null-value">null</a></td>
<td> 1-byte</td>
<td> Z</td>
<td> No</td>
<td> No</td>
</tr>
<tr>
<td><a href="value-types#no-op-value">no-op</a></td>
<td> 1-byte</td>
<td> N</td>
<td> No</td>
<td> No</td>
</tr>
<tr>
<td><a href="value-types#boolean-types">true</a></td>
<td> 1-byte</td>
<td> T</td>
<td> No</td>
<td> No</td>
</tr>
<tr>
<td><a href="value-types#boolean-types">false</a></td>
<td> 1-byte</td>
<td> F</td>
<td> No</td>
<td> No</td>
</tr>
<tr>
<td><a href="value-types#numeric-types">int8</a></td>
<td> 2-bytes</td>
<td> i</td>
<td> No</td>
<td> Yes</td>
</tr>
<tr>
<td><a href="value-types#numeric-types">uint8</a></td>
<td> 2-bytes</td>
<td> U</td>
<td> No</td>
<td> Yes</td>
</tr>
<tr>
<td><a href="value-types#numeric-types">int16</a></td>
<td> 3-bytes</td>
<td> I</td>
<td> No</td>
<td> Yes</td>
</tr>
<tr>
<td><a href="value-types#numeric-types">int32</a></td>
<td> 5-bytes</td>
<td> l</td>
<td> No</td>
<td> Yes</td>
</tr>
<tr>
<td><a href="value-types#numeric-types">int64</a></td>
<td> 9-bytes</td>
<td> L</td>
<td> No</td>
<td> Yes</td>
</tr>
<tr>
<td><a href="value-types#numeric-types">float32</a></td>
<td> 5-bytes</td>
<td> d</td>
<td> No</td>
<td> Yes</td>
</tr>
<tr>
<td><a href="value-types#numeric-types">float64</a></td>
<td> 9-bytes</td>
<td> D</td>
<td> No</td>
<td> Yes</td>
</tr>
<tr>
<td><a href="value-types#numeric-types">high-precision number</a></td>
<td> 1-byte + int num val + string byte len</td>
<td> H</td>
<td> Yes</td>
<td> Yes (if non-empty)</td>
</tr>
<tr>
<td><a href="value-types#char-type">char</a></td>
<td> 2-bytes</td>
<td> C</td>
<td> No</td>
<td> Yes</td>
</tr>
<tr>
<td><a href="value-types#string-type">string</a></td>
<td> 1-byte + int num val + string byte len</td>
<td> S</td>
<td> Yes</td>
<td> Yes (if non-empty)</td>
</tr>
<tr>
<td colspan="5">

> Container Types

</td>
</tr>
<tr>
<td><a href="container-types#array-type">array</a>**</td>
<td> 2+ bytes</td>
<td> [ and ]</td>
<td> Optional</td>
<td> Yes (if non-empty)</td>
</tr>
<tr>
<td><a href="container-types#object-type">object</a>**</td>
<td> 2+ bytes</td>
<td> { and }</td>
<td> Optional</td>
<td> Yes (if non-empty)</td>
</tr>
</tbody>
</table>
<sup>** See <a href="container-types#optimized-format">container optimized format</a> for details.</sup>

## Example

Below is an example of what a common JSON response would look like in UBJSON. This particular example was taken from the [GitHub developer docs](http://developer.github.com/v3/users/).

**JSON Response**
```json
{
  "login": "octocat",
  "id": 1,
  "avatar_url": "https://github.com/images/error/octocat_happy.gif",
  "gravatar_id": "somehexcode",
  "url": "https://api.github.com/users/octocat",
  "name": "monalisa octocat",
  "company": "GitHub",
  "blog": "https://github.com/blog",
  "location": "San Francisco",
  "email": "octocat@github.com",
  "hireable": false,
  "bio": "There once was...",
  "public_repos": 2,
  "public_gists": 1,
  "followers": 20,
  "following": 0,
  "html_url": "https://github.com/octocat",
  "created_at": "2008-01-14T04:33:35Z",
  "type": "User",
  "total_private_repos": 100,
  "owned_private_repos": 100,
  "private_gists": 81,
  "disk_usage": 10000,
  "collaborators": 8,
  "plan": {
    "name": "Medium",
    "space": 400,
    "collaborators": 10,
    "private_repos": 20
  }
}
```
**UBJSON Response (using block-notation)**
```
[{]
    [i][5][login][S][i][7][octocat]
    [i][2][id][i][1]
    [i][10][avatar_url][S][i][49][https://github.com/images/error/octocat_happy.gif]
    [i][11][gravatar_id][S][i][11][somehexcode]
    [i][3][url][S][i][36][https://api.github.com/users/octocat]
    [i][4][name][S][i][16][monalisa octocat]
    [i][7][company][S][i][6][GitHub]
    [i][4][blog][S][i][23][https://github.com/blog]
    [i][8][location][S][i][13][San Francisco]
    [i][5][email][S][i][18][octocat@github.com]
    [i][8][hireable][F]
    [i][3][bio][S][i][17][There once was...]
    [i][12][public_repos][i][2]
    [i][12][public_gists][i][1]
    [i][9][followers][i][20]
    [i][9][following][i][0]
    [i][8][html_url][S][i][26][https://github.com/octocat]
    [i][10][created_at][S][i][20][2008-01-14T04:33:35Z]
    [i][4][type][S][i][4][User]
    [i][19][total_private_repos][i][100]
    [i][19][owned_private_repos][i][100]
    [i][13][private_gists][i][81]
    [i][10][disk_usage][I][10000]
    [i][13][collaborators][i][8]
    [i][4][plan][{]
        [i][4][name][S][i][6][Medium]
        [i][5][space][I][400]
        [i][13][collaborators][i][10]
        [i][13][private_repos][i][20]
    [}]
[}]
```