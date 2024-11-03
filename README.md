
# GitHub Users in Tokyo

This repository contains data and analysis on GitHub users based in Tokyo with over 200 followers, along with information about their public repositories.

## Files

1. **`users.csv`**: Contains data on 545 GitHub users from Tokyo with over 200 followers.
2. **`repositories.csv`**: Contains data on 67,148 public repositories from these users.
3. **`data_scrap.py`**: Python script used to collect the data.
4. **`Project-1-Analysis.ipynb`**: Colab notebook containing Python code and analysis results for 16 questions in Project 1.

## Data Collection

- **Source**: GitHub API
- **Collection Date**: 2024-10-30
- **Criteria**: Only users with over 200 followers from Tokyo are included.
- **Repository Limit**: Up to 500 of the most recently pushed repositories per user.

## Data Scraping Process

Data was collected using the GitHub API with a specific query to filter users based on location (Tokyo) and follower count (200+). Below is a breakdown of the scraping process.

### 1. Authorization and Headers

The script sets up an authorization token to bypass stricter rate limits for unauthenticated access.

```python
GITHUB_TOKEN = 'my_access_token'
HEADERS = {"Authorization": f"token {GITHUB_TOKEN}"}
```

### 2. Helper Function for Cleaning Company Names

The `clean_company_name` function standardizes company names by removing any leading "@" symbols, trimming whitespace, and converting to uppercase.

```python
def clean_company_name(company):
    if company:
        company = company.strip().lstrip('@').upper()
    return company
```

### 3. Fetching User Data

The `fetch_users` function retrieves users from Tokyo with at least 200 followers. It fetches user data page by page, pausing after each page to avoid rate limits. Full user details, such as login, name, company, email, and other fields, are retrieved for each user.

```python
def fetch_users(city="Tokyo", min_followers=200):
    users = []
    page = 1
    while True:
        url = f"https://api.github.com/search/users?q=location:{city}+followers:>{min_followers}&page={page}&per_page=100"
        response = requests.get(url, headers=HEADERS)
        data = response.json()
        
        for user in data['items']:
            user_url = user['url']
            user_response = requests.get(user_url, headers=HEADERS)
            user_data = user_response.json()
            users.append({
                'login': user_data['login'],
                'name': user_data['name'],
                'company': clean_company_name(user_data['company']),
                'location': user_data['location'],
                'email': user_data['email'],
                'hireable': user_data['hireable'],
                'bio': user_data['bio'],
                'public_repos': user_data['public_repos'],
                'followers': user_data['followers'],
                'following': user_data['following'],
                'created_at': user_data['created_at'],
            })
        page += 1
        time.sleep(1)
```

### 4. Fetching Repository Data

The `fetch_repositories` function retrieves all public repositories for each user. It collects details such as repository name, creation date, stargazers, and license type.

```python
def fetch_repositories(user_login):
    repositories = []
    page = 1
    while True:
        url = f"https://api.github.com/users/{user_login}/repos?per_page=100&page={page}"
        response = requests.get(url, headers=HEADERS)
        repo_data = response.json()
        
        for repo in repo_data:
            repositories.append({
                'login': user_login,
                'full_name': repo['full_name'],
                'created_at': repo['created_at'],
                'stargazers_count': repo['stargazers_count'],
                'watchers_count': repo['watchers_count'],
                'language': repo['language'],
            })
        if len(repo_data) < 100:
            break
        page += 1
        time.sleep(1)
```

### 5. Saving Data to CSV

The functions `save_users_to_csv` and `save_repositories_to_csv` save user and repository data to CSV files, respectively.

```python
def save_users_to_csv(users, filename="users.csv"):
    with open(filename, mode="w", newline="", encoding="utf-8") as file:
        writer = csv.DictWriter(file, fieldnames=users[0].keys())
        writer.writeheader()
        writer.writerows(users)

def save_repositories_to_csv(repositories, filename="repositories.csv"):
    with open(filename, mode="w", newline="", encoding="utf-8") as file:
        writer = csv.DictWriter(file, fieldnames=repositories[0].keys())
        writer.writeheader()
        writer.writerows(repositories)
```

### 6. Main Execution

The main function orchestrates the entire process, fetching users, saving them to `users.csv`, then fetching and saving repositories to `repositories.csv`.

```python
def main():
    print("Fetching users...")
    users = fetch_users()
    save_users_to_csv(users)
    print(f"Saved {len(users)} users to users.csv")

    print("Fetching repositories...")
    all_repositories = []
    for user in users:
        user_repos = fetch_repositories(user["login"])
        all_repositories.extend(user_repos)
        print(f"Fetched {len(user_repos)} repositories for user {user['login']}")

    save_repositories_to_csv(all_repositories)
    print(f"Saved {len(all_repositories)} repositories to repositories.csv")

if __name__ == "__main__":
    main()
```

## Key Insights from Analysis

1. **Top 5 Users by Followers**: `dennybritz`, `wasabeef`, `dai-shi`, `rui314`, `domenic`
2. **Earliest Registered Users**: `kana`, `kakutani`, `mootoh`, `lhl`, `walf443`
3. **Popular Licenses**: `MIT`, `Apache-2.0`, and others
4. **Top Company**: Most developers work at **Google**
5. **Popular Languages**: 
   - Most popular: **JavaScript**
   - For users who joined after 2020: **Rust**
   - Highest average stars: **Assembly**
6. **Top 5 by Leader Strength**: `blueimp`, `dai-shi`, `asahilina`, `pilcrowonpaper`, `marcan`
7. **Correlation**: A slight positive correlation (0.051) between follower count and number of repositories.
8. **Additional Findings**:
   - Hireable users often share their email addresses.
   - Users with bio tend to have higher visibility.


## Some Interesting Findings from users.csv data :

**1.Influential users :**

Some of the users are considered influential or popular based on their large followers and public_repos.

**2.Inactive/Unhireable users :**

Although inactive/unhireable users also make up one-third of total users, so many of its profiles do not look at new chances.

**3.Geo Distribution :**

The location field describes many users based in major tech hubs, which can offer potential insight into distributions of tech ecosystems.

**4.Users Bio :**

One of the most startling findings of this analysis was that the developers with very short and precise bios received more followers than those developers with very long and elaborate descriptions.

## Some Interesting Findings from repository.csv data :

**1.Repositories Popularity :**

The average number of stars by a stargazer is zero, and the most of the repositories have a minimal count. However, 25% of the repositories have at least 2 stars, and the most popular repository has 58,479 stars.

**2.Most popular programming languages :**

JavaScript (16.7%), Ruby (10%), Python (7.8%), Go (6.4%), and TypeScript (6.3%). This is a point of showing that developers typically make use of flexible web-centric languages, such as JavaScript and TypeScript.

**3.License Preferences :**

Most of the repositories (54.5%) use MIT License, possibly because of their permissive nature. Among other licenses are Apache 2.0 (15%), and some GNU as well as BSD licenses.

## Action Recommendations

### For Users

- **Repository Visibility**: Engage more with the JavaScript or other popular language communities. Adding documentation and projects can increase repository visibility.
- **Networking**: Connect with highly followed users to expand networks.

### For Community Growth

- **Encourage Collaboration**: Since most repositories have projects and wikis enabled, promoting open-source contributions can help foster a more engaged developer community.

