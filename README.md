# GitHub Users in Tokyo

This repository contains data about GitHub users in Tokyo with over 200 followers and their repositories.

## Files

1. `users.csv`: Contains information about 545 GitHub users in Tokyo with over 200 followers
2. `repositories.csv`: Contains information about 67148 public repositories from these users
3. `data_scrap.py`: Python script used to collect this data
4. `Project-1-Analysis.ipynb`: Colab notebook contains python code and analysis result for all 16 questions in project 1

## Data Collection

- Data collected using GitHub API
- Date of collection: 2024-10-30
- Only included users with 200+ followers
- Up to 500 most recently pushed repositories per user


## Explanation about how scrapping is done :

The scrapping process is done using Github API's end point "https://api.github.com/search/users?q=location:Tokyo+followers:>200&page={page}&per_page=100" to get a list of all users that are from Tokyo and have more than 200 followers. Then "https://api.github.com/user/{user_id}" is used to get all the required details of the users. For repositories, "https://api.github.com/users/{user_login}/repos?per_page=100&page={page}" is used to get details about each repository present for every user, storing all the data in a csv file using python.

The explaination for the python code is given below:

1.Authorization and Headers :

This code sets up the GitHub API token and headers for authentication, bypassing the stricter rate limits on unauthenticated access.

Code :

  GITHUB_TOKEN = 'my_access_token'
  HEADERS = {"Authorization": f"token {GITHUB_TOKEN}"}
2. Helper function to clean company names :

'the clean_company_name' function removes the leading "@" sign, removes leading and trailing white space and forces to upper case.

Code :

  def clean_company_name(company):
      if company:
          company = company.strip().lstrip('@').upper()
      return company
3.FETCHING USER DATA :

'fetch_users' function fetches users in a given city (default: Tokyo) with at least a specified follower count from the GitHub API. In fact it fetches the users page by page until there are no more results, pausing for a second after every page to avoid rate limits. For each of the found users, the user's full details such as login, name, location, company, email, etc., are retrieved with an additional API request.

Code :

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
4.Repository Data Fetching :

'fetch_repositories' function fetches from the GitHub API all the public repositories of each user, page by page. Details drawn will include name, date created, number of stargazers, and license per repository. The function returns a list of repositories related to the user, and every type of data is stored in dictionaries.

Code :

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
                  'language': repo
5.Saving Data to CSV :

'save_users_to_csv' and 'save_repositories_to_csv' saves user and repository information to two different CSVs. DictWriter is used to ensure each dictionary's keys map to the CSV headers.

Code :

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
6.Main Execution (main function) :

This orchestrates the entire process: fetch the user, save them into a file named users.csv; fetch all repositories from these users, save that to repositories.csv.

Code :

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


# Key Insights from Data (....Analysis....)

"dennybritz,wasabeef,dai-shi,rui314,domenic" are top 5 users

"kana,kakutani,mootoh,lhl,walf443" are the 5 earliest registered GitHub users in 'Tokyo'

"mit,apache-2.0,other" are the 3 most popular license among these users

Most of the developers works at "GOOGLE"

"Javascript" is the programming language that is most popular among these users

"Rust" is the programming language that is the second most popular among users who joined after '2020'

"Assembly" language has the highest average number of stars per repository

"blueimp,dai-shi,asahilina,pilcrowonpaper,marcan" are the top 5 in terms of 'leader_strength'

The correlation between the number of followers and the number of public repositories among users in Tokyo is "0.051"

Creating more repos help users get more followers

People typically enable projects and wikis together

Hireable users don't follow more people than those who are not hireable

Positive impact of the length of their bio

"azu,suzuki-shunsuke,yuiseki,xuwei-k,zchee" are the top 5 users who created the most repositories on weekends

People who are hireable share their email addresses more often

"Kato,Tanaka" are most common surname



# Action Recommendations

## For Users:

Improve Repository Visibility: Users can increase their repository stars and watchers by engaging more with the JavaScript community or other popular language communities. Including more documentation and projects can also boost visibility.
Networking Opportunities: Users can connect with highly followed individuals (e.g., top 5 users) to expand their own network.


## For Community Growth:

Encourage Collaboration: Since most repositories have projects and wikis enabled, promoting collaborative efforts, such as open-source contributions, can foster a more engaged developer community.
