



```js

App.currentUser.whenAuthenticated().then(function () {
	App.serverModules.fetchModule('GO\\Modules\\GroupOffice\\Imap\\Module').then(function (module) {

		App.addAccountType('GO\\Modules\\GroupOffice\\Imap\\Model\\Account', 'GO.Modules.GroupOffice.Imap.Model.Account', 'email', {
			templateUrl: 'modules/groupoffice/imap/views/account.html',
			controller: 'GO.Modules.GroupOffice.Imap.Controller.Account',
			inputs: {
				autodetect: null //Create account dialog passes an autodetect model. We must declare it here too for the Dep inject.
			}
		}, {
			templateUrl: 'modules/groupoffice/imap/views/auto-detect.html',
			controller: 'GO.Modules.GroupOffice.Imap.Controller.AutoDetect'
		});
	});
});

```