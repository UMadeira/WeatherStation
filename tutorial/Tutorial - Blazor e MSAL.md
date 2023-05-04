## Autenticação através da biblioteca MSAL
O objetivo deste tutorial é ilustrar a utilização da biblioteca **MSAL** (Microsoft Authentication Library)para autentificar os utilizadores, evitando a necessidade implementar um mecanismo de autenticação e gestão de utilizadores.

### Prérequisitos

Para a realização deste tutorial é necessário o seguinte:

* Visual Studio 2022
* Uma conta Microsoft com acesso ao portal Azure (https://portal.azure.com)

### Autenticação de uma aplicação cliente usando o Blazor 

O Blazor pode ser utilizado para criar aplicações servidor (server side) ou aplicações cliente (client side). Neste tutorial, vamos começar por criar uma aplicação cliente.

#### Utilizando o CLI

1. Crie uma diretoria para a solução

   ```powershell
   mkdir WeatherStation
   cd .\WeatherStation
   ```

2. Crie uma solução em branco na diretoria 

   ```powershell
   dotnet new sln --name WeatherStation
   ```

3. Crie uma aplicação Blazor WebAssembly

   ```powershell
   dotnet new blazorwasm -n WeatherStationApp -au SingleOrg
   ```

4. Adicione o projeto à solução
   ```powershell
   dotnet sln .\WeatherStation.sln add .\WeatherStationApp\WeatherStationApp.csproj
   ```

#### Utilizando o Visual Studio 2022

Brevemente...

### Registar a aplicação no portal Azure

Para aceder ao portal da Azure (https://portal.azure.com) é necessário ter uma conta individual da Microsoft. 

1. Obter o URL da aplicação web (no presente caso ``https://localhost:7016`` )

   ![2.0 - Azure](https://raw.githubusercontent.com/UMadeira/WeatherStation/edit/develop/tutorial/images/2.0-Azure.png)

2. Aceder ao portal da Azure (https://portal.azure.com)

3. Selecionar o serviço "App registrations"

   ![2.1 - Azure](./images/2.1 - Azure.png)

4. Criar um novo registo

   ![2.2 - Azure](./images/2.2 - Azure.png)

5. Preencher o registo com a seguinte informação (não esquecer de colocar ``/authentication/login-callback`` após o URL da aplicação)

   ![2.3 - Azure](./images/2.3 - Azure.png)

6. Recolher os valores de **ClientId** (``eef826f7-58ce-4817-806d-55462a7a63f3``) e do **TenantId** (`f8cdef31-a31e-4b4a-93e4-5f571e91255a`). Navegar para a opção  **Integration assistant**.

   ![2.4 - Azure](./images/2.4 - Azure.png)

7. Selecionar as opções **Single-page app** e **Web API** (posteriormente será utilizado este tipo de integração)

   ![2.5.1 - Azure](./images/2.5.1 - Azure.png)

8. Selecionar **Evaluate my app registration**

   ![2.5 - Azure](./images/2.5 - Azure.png)

9. Na página do **Integration Assistant**, na opção **Provide links to your app's terms of service and privacy statement** selecionar **Go to page**

   ![2.6 - Azure](./images/2.6 - Azure.png)

10. Configurar com informação análoga à apresentada na seguinte figura e fazer **Save**

    ![2.7 - Azure](./images/2.7 - Azure.png)

    ### Configurar a aplicação com os  valores de registo

    1. Editar o ficheiro ``appsettings.json`` e colocar os valores de ``"ClientId"`` e ajustar o valor de ``"Authority"``

       ![4.0 - Configure](./images/4.0 - Configure.png)

    2. No ficheiro ``Program.cs`` adicionar a seguinte configuração para permitir o acesso aos dados de perfil do utilizador ("https://graph.microsoft.com/User.Read")

       ```c#
       using Microsoft.AspNetCore.Components.Web;
       using Microsoft.AspNetCore.Components.WebAssembly.Hosting;
       using WeatherStationApp;
       
       var builder = WebAssemblyHostBuilder.CreateDefault(args);
       builder.RootComponents.Add<App>("#app");
       builder.RootComponents.Add<HeadOutlet>("head::after");
       
       builder.Services.AddScoped(sp => new HttpClient { 
           BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) } );
       
       builder.Services.AddMsalAuthentication( options =>
       {
           options.ProviderOptions.AdditionalScopesToConsent.Add(
               "https://graph.microsoft.com/User.Read" );
       
           builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
       });
       
       await builder.Build().RunAsync();
       ```

    3. Realizar alguma "reengenharia" do código presente na página ``FetchData.razor``

       a. Crie uma diretoria **Data** no projeto e mova o objeto **WeatherForecast** para essa diretoria.

          ![4.1 - Configure](./images/4.1 - Configure.png)

       b. Crie uma pasta **Components** e crie uma componente Blazor designada por ``WeatherForecastTable.razor``

          ![4.2 - Configure](./images/4.2 - Configure.png)

       c. Edite a página ``FetchData.razor``  de forma a proteger a view

       ![4.3 - Configure](./images/4.3 - Configure.png)
    
    4. Execute e teste

