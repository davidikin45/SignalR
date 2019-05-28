# SignalR
* Uses Web Sockets rather than HTTP Requests
* Two-Way Communication
* Web Sockets faster than HTTP
* If Web Sockets isn't enabled can use Server Side Events (SSE) or Long Polling.
* SignalR is an open source framework that wraps the complexity of real-time web transports.
* Web Sockets > Server Side Events (SSE) > Long Polling
* Remote Procedure Calls (RPC) calls between Browser > Server and Server > Browser
* A Real-time web application is the an application that supports server to client communication
* Any methods within the Hub can be called from client javascript.
* a hub is a server-side class that sends messages to and receives messages from clients
* npm install @aspnet/signalr
* Ensure Web Sockets are enabled on Azure. Settings > Configuration > Web Sockets
* Ensure Web Sockets are enabled on IIs. Windows Features > World Wide Web Services > Application Development Features > WebSocket Protocol
* If Web Sockets are not enabled SignalR will use Server Side Events (SSE)
* Azure SignalR Service takes care of scaling without needing to run multiple instances of application.
* [Getting Started with ASP.NET Core SignalR](https://app.pluralsight.com/library/courses/aspdotnet-core-signalr-getting-started/table-of-contents)

## Server Example
```
public interface IChatClient
{
	Task ReceiveMessage(string message);
}

public class ChatHub : Hub<IChatClient>
{
	//Client JS RPC methods

	public Task SendMessageToClients(string message, params string[] connectionIds)
	{
		//return Clients.Clients(connectionIds.ToList()).SendAsync("ReceiveMessage", message);
		return Clients.Clients(connectionIds.ToList()).ReceiveMessage(message);
	}

	public Task SendMessageToUsers(string message, params string[] userIds)
	{
		//return Clients.Users(userIds.ToList()).SendAsync("ReceiveMessage", message);
		return Clients.Users(userIds.ToList()).ReceiveMessage(message);
	}

	public Task SendMessageToAllUsers(string message)
	{
		//return Clients.All.SendAsync("ReceiveMessage", message);
		return Clients.All.ReceiveMessage(message);
	}

	public Task SendMessageToGroups(string message, params string[] groups)
	{
		//return Clients.Groups(groups.ToList()).SendAsync("ReceiveMessage", message);
		return Clients.Groups(groups.ToList()).ReceiveMessage(message);
	}
	public Task SendMessageBackToSender(string message)
	{
		//return Clients.Caller.SendAsync("ReceiveMessage", message);
		return Clients.Caller.ReceiveMessage(message);
	}

	public override async Task OnConnectedAsync()
	{

		var roles = Context.User.Claims.Where(c => c.Type == ClaimTypes.Role || c.Type == "role")
				   .Select(c => c.Value)
				   .ToList();

		foreach (var role in roles)
		{
			await Groups.AddToGroupAsync(Context.ConnectionId, role);
		}

		await base.OnConnectedAsync();
	}

	public override async Task OnDisconnectedAsync(Exception exception)
	{
		var roles = Context.User.Claims.Where(c => c.Type == ClaimTypes.Role || c.Type == "role")
				 .Select(c => c.Value)
				 .ToList();

		foreach (var role in roles)
		{
			await Groups.RemoveFromGroupAsync(Context.ConnectionId, role);
		}

		await base.OnDisconnectedAsync(exception);
	}
}

public static class ChatHubServerExtensions
{
	public static Task SendMessageToClients(this IHubContext<ChatHub, IChatClient> hubContext, string message, params string[] connectionIds)
	{
		//return hubContext.Clients.Clients(connectionIds.ToList()).SendAsync("ReceiveMessage", message);
		return hubContext.Clients.Clients(connectionIds.ToList()).ReceiveMessage(message);
	}

	public static Task SendMessageToUsers(this IHubContext<ChatHub, IChatClient> hubContext, string message, params string[] userIds)
	{
		//return hubContext.Clients.Users(userIds.ToList()).SendAsync("ReceiveMessage", message);
		return hubContext.Clients.Users(userIds.ToList()).ReceiveMessage(message);
	}

	public static Task SendMessageToAllUsers(this IHubContext<ChatHub, IChatClient> hubContext, string message)
	{
		//return hubContext.Clients.All.SendAsync("ReceiveMessage", message);
		return hubContext.Clients.All.ReceiveMessage(message);
	}

	public static Task SendMessageToGroups(this IHubContext<ChatHub, IChatClient> hubContext, string message, params string[] groups)
	{
		//return hubContext.Clients.Groups(groups.ToList()).SendAsync("ReceiveMessage", message);
		return hubContext.Clients.Groups(groups.ToList()).ReceiveMessage(message);
	}
}
```

## Client Example
```
var signalRNotificationsConnection = null;

function setupSignalRNotificationsConnection()
{
    signalRNotificationsConnection = new signalR.HubConnectionBuilder()
        .withUrl("/api/signalR/notifications")
        .build();

    signalRNotificationsConnection.on("ReceiveMessage", function (message) {
        alert(message);
    }
    );

    signalRNotificationsConnection.on("Finished", function () {
        connection.stop();
    }
    );

    signalRNotificationsConnection.start()
        .catch(function (err) {
            console.error(err.toString());
        });
};

setupSignalRNotificationsConnection();
```