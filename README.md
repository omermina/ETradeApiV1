# ETradeApiV1

Rest client library for using the E*Trade V1 new API Supports both **.NET Framework** and **.NET Core** (by using .NET Standard)!

## Install
Install package from [NuGet](https://www.nuget.org/packages/ETradeApi/) or Package Manager Console:

`PM> Install-Package ETradeApi`

## How do I use this library?

### App.Config
```
<appSettings>
    <add key="BaseUrl" value="https://apisb.etrade.com/v1/" />
    <add key="TokenUrl" value="https://api.etrade.com/" />
    <add key="AuthorizeUrl" value="https://us.etrade.com/e/t/etws/" />
    <add key="ConsumerKey" value="Your_ConsumerKey" />
    <add key="ConsumerSecret" value="Your_ConsumerSecret" />
    <add key="AccessSecret" value="" />
    <add key="AccessToken" value="" />
  </appSettings>
```
  
### Console Application Example
```
class Program
    {
        private static EtApiService _apiServices;
        static void Main(string[] args)
        {
            string key;
            do
            {
                System.Console.WriteLine("1. Authenticate  ");
                System.Console.WriteLine("2. Get Quote");
                System.Console.WriteLine("3. Get Accounts List ");
                System.Console.WriteLine("Enter key: ");

                key = System.Console.ReadLine();
            }
            while (key == null);

            switch (key)
            {
                case "1":
                    Authenticate_Etrade_With_Client();
                    break;
                case "2":
                    GetQuote();
                    break;
                case "3":
                    GetAccountsList();
                    break;

            }


        }

        private static void Authenticate_Etrade_With_Client()
        {

            var config = EtConfigurationService.GetOAuthConfigFromSetting();
            _apiServices = new EtApiService(config);

            var authorizeUrl = _apiServices.GetAuthorizeUrl();
            Process.Start(authorizeUrl);

            string verificationKey;

            do
            {
                System.Console.Write("Enter verification key: ");

                verificationKey = System.Console.ReadLine();
            }
            while (verificationKey == null);

            var isSet = _apiServices.SetAccessToken(verificationKey);
            if (isSet) EtConfigurationService.SaveTokenTpConfig(config);


        }

        private static void GetQuote()
        {
            var config = EtConfigurationService.GetOAuthConfigFromSetting();
            var response = EtApiService.GetQuote(config,"CVS,T");

            foreach (var item in response.Data.QuoteResponse.QuoteData)
            {
                System.Console.WriteLine($"{item.Product.symbol} {item.Product.securityType}");
            }

            System.Console.ReadLine();
        }


        private static void GetAccountsList()
        {
            var config = EtConfigurationService.GetOAuthConfigFromSetting();
            var qClient = new RestClient
            {
                BaseUrl = new Uri(config.BaseUrl),
                Authenticator = OAuth1Authenticator.ForProtectedResource(config.ConsumerKey, config.ConsumerSecret, config.AccessToken, config.AccessSecret)

            };

            var accountsListRequest = new RestRequest("accounts/list");
            var response = qClient.Execute<AccountsListDto>(accountsListRequest);

            foreach (var item in response.Data.AccountListResponse.Accounts.Account)
            {
                System.Console.WriteLine($"{item.AccountName} {item.AccountStatus}");
            }

            System.Console.ReadLine();
        }

    }
```
