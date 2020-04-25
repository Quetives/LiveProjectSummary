# LiveProjectSummary
Summary of the LiveProject

For the last two weeks of my time at the tech academy, I worked with my peers in a team developing a full scale MVC/MVVM Web Application in C#. Working on a legacy codebase was a great learning oppertunity for fixing bugs, cleaning up code and adding requested features. There was alot of work that needed to be implemented, but the team worked together to produce many new features. Our team was utilizing the Scrum Framework and I was able to get experience with this method of production. I was working on a project that was already half built. I worked on several back end/front end stories all of varying degrees of difficulty. Everyone on the team had a chance to work on front end and back end stories as they chose. Over the two week sprint I also had the opportunity to work on some project management and team development skills that I'm confident I will use in future projects.

Below are descriptions of the stories I worked on, along with code snippets and navigation links. I also have some full code files in this repo for the larger functionalities I implemented.

# Back End Stories

# Display Photo Null or Invalid ID handling

There was a method in the photo class called DisplayPhoto that would currently throw an error if it was provided a null or invalid id. Accounting for these situations I researched how the project was using byteArrays to display photos and came up with this solution.

 public ActionResult DisplayPhoto(int? id) //nullable int
        {            
            var byteData = new byte[] { 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20 };           
                if (id.HasValue)
                {                
                Photo photo = db.Photo.Find(id);
                    if (photo == null)
                    {
                        return File(byteData, "image/png");
                    }
                    else
                    {
                        return File(photo.PhotoFile, "image/png");
                    }                                                                  
                }
                else
                { 
                    return File(byteData, "image/png");
                }
            }

However soon after I realized how I could refactor my code:

 public ActionResult DisplayPhoto(int? id) //nullable int
        {            
            var byteData = new byte[] { 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20 };           
                if (id.HasValue)
                {                
                Photo photo = db.Photo.Find(id);
		    if (photo != null)
		{
			return File(photo.PhotoFile, "image/png");
		}
		else
		{
			return File(byteData, "image/png");
		}
	    }



# Implementing Email Confirmation For New Accounts

This was the most difficult part of the project for me. The instructions for this wanted me to implement the SendGrid package and utilize their service to generate and send out confirmation tokens. I gained alot of knowledge working with 3rd party services on this story. I had to create my own SendGrid account and generate an API key to be able to test and use this service. Once I generated the key I put it into an Environmental Variable to be called upon by the project.


Within the Register function in the AccountController I added a call to my own function.

await EmailCodeSendGrid(user, model);
                    ViewBag.Message = "Check your email and confirm your account, you must be confirmed before you can log in";
                    return View("Info");

This would call my function:

[HttpPost]
        public async Task<ActionResult> EmailCodeSendGrid(ApplicationUser user, RegisterViewModel model)
        {            
                string code = await UserManager.GenerateEmailConfirmationTokenAsync(user.Id);
                var callbackUrl = Url.Action("ConfirmEmail", "Account", new { userId = user.Id, code = code }, protocol: Request.Url.Scheme);
                string message = "Please confirm your account by clicking <a href=\"" + callbackUrl + "\">here</a>";
                string subject = "Confirm your account";
                var apiKey = Environment.GetEnvironmentVariable("SendGrid_Key");
                var client = new SendGridClient(apiKey);
                var msg = new SendGridMessage()
                
                {
                    From = new EmailAddress("r.quetives@gmail.com", "Richard Quetives"),
                    Subject = subject,
                    PlainTextContent = message,
                    HtmlContent = message
                };
                msg.AddTo(new EmailAddress(model.Email, model.FirstName + " " + model.LastName));
                var response = await client.SendEmailAsync(msg);

                // Uncomment to debug locally
                //TempData["ViewBagLink"] = callbackUrl;
                bool confirm = (Convert.ToString(response.StatusCode) == "Accepted") ? true : false;
                return Json(confirm);
        }





# Render different CurrentMember Detail layout based on user roles

For this task the project manager did not want the edit buttons on the Members Details page to appear unless you had were in the Admin user role. So for this I implemented a simple check of the user role before displaying the edit button.

<p>
    @if (User.IsInRole("Admin"))
    {
        @Html.ActionLink("Edit", "Edit", new { id = Model.CastMemberID })
        @(" |")
    }
    @Html.ActionLink("Back to List", "Index")
</p>

This had the desired effect of hiding the edit button for all but the Admins and cleanly displayed the Back to List for everyone.





# Front End Stories

Subscriber Details page

During this project I wanted to do at least one front end story.

Previously the Subscriber Details page was basically just a list of subscriber details with no styling whatsoever:

@*<div>
        <h4>Subscriber</h4>
        <hr />
        <dl class="dl-horizontal">
            <dt>
                <label class="font-weight-bold h5">@(ViewBag.User)</label>
            </dt>
            <br />
            <dt>
                @Html.DisplayNameFor(model => model.CurrentSubscriber)
            </dt>

            <dd>
                @Html.DisplayFor(model => model.CurrentSubscriber)
            </dd>

            <dt>
                @Html.DisplayNameFor(model => model.HasRenewed)
            </dt>

            <dd>
                @Html.DisplayFor(model => model.HasRenewed)
            </dd>

            <dt>
                @Html.DisplayNameFor(model => model.Newsletter)
            </dt>

            <dd>
                @Html.DisplayFor(model => model.Newsletter)
            </dd>

            <dt>
                @Html.DisplayNameFor(model => model.RecentDonor)
            </dt>

            <dd>
                @Html.DisplayFor(model => model.RecentDonor)
            </dd>

            <dt>
                @Html.DisplayNameFor(model => model.LastDonated)
            </dt>

            <dd>
                @Html.DisplayFor(model => model.LastDonated)
            </dd>

            <dt>
                @Html.DisplayNameFor(model => model.LastDonationAmt)
            </dt>

            <dd>
                @Html.DisplayFor(model => model.LastDonationAmt)
            </dd>

            <dt>
                @Html.DisplayNameFor(model => model.SpecialRequests)
            </dt>

            <dd>
                @Html.DisplayFor(model => model.SpecialRequests)
            </dd>

            <dt>
                @Html.DisplayNameFor(model => model.Notes)
            </dt>

            <dd>
                @Html.DisplayFor(model => model.Notes)
            </dd>

        </dl>
    </div>*@

To fix this I used bootstrap to put all of these items in a list-group and created some custom css styling that I used to fit the color scheme and formatting of this project.


@{
    ViewBag.Title = "Details";
    ViewBag.User = Model.SubscriberPerson.FirstName + " " + Model.SubscriberPerson.LastName;
}

<body class="subbody pt-5 sub-detail-bg">
    <div class="container pb-1 subback1 w-75 sub-row-content mb-4">
        <h2>Subscriber Details</h2>
    </div>

    <div class="container subback1 w-75">
        <div class="list-group list-group-flush" style="border: none">

            <label class="font-weight-bold h3">@(ViewBag.User)</label>

            <div class="list-group-item rounded-top border-0">
                <h5 class="list-group-item-heading subback2">@Html.DisplayNameFor(model => model.CurrentSubscriber)</h5>
                <p class="list-group-item-text">@Html.DisplayFor(model => model.CurrentSubscriber)</p>
            </div>

            <div class="list-group-item border-0">
                <h5 class="list-group-item-heading subback2">@Html.DisplayNameFor(model => model.HasRenewed)</h5>
                <p class="list-group-item-text">@Html.DisplayFor(model => model.HasRenewed)</p>
            </div>

            <div class="list-group-item border-0">
                <h5 class="list-group-item-heading subback2">@Html.DisplayNameFor(model => model.Newsletter)</h5>
                <p class="list-group-item-text">@Html.DisplayFor(model => model.Newsletter)</p>
            </div>

            <div class="list-group-item border-0">
                <h5 class="list-group-item-heading subback2">@Html.DisplayNameFor(model => model.RecentDonor)</h5>
                <p class="list-group-item-text">@Html.DisplayFor(model => model.RecentDonor)</p>
            </div>

            <div class="list-group-item border-0">
                <h5 class="list-group-item-heading subback2">@Html.DisplayNameFor(model => model.LastDonated)</h5>
                <p class="list-group-item-text list-group-item-action">@Html.DisplayFor(model => model.LastDonated)</p>
            </div>

            <div class="list-group-item border-0">
                <h5 class="list-group-item-heading subback2">@Html.DisplayNameFor(model => model.LastDonationAmt)</h5>
                <p class="list-group-item-text list-group-item-action">@Html.DisplayFor(model => model.LastDonationAmt)</p>
            </div>

            <div class="list-group-item border-0">
                <h5 class="list-group-item-heading subback2">@Html.DisplayNameFor(model => model.SpecialRequests)</h5>
                <p class="list-group-item-text list-group-item-action">@Html.DisplayFor(model => model.SpecialRequests)</p>
            </div>

            <div class="list-group-item rounded-bottom border-0">
                <h5 class="list-group-item-heading subback2">@Html.DisplayNameFor(model => model.Notes)</h5>
                <p class="list-group-item-text list-group-item-action">@Html.DisplayFor(model => model.Notes)</p>
            </div>
        </div>
        <p>
            @Html.ActionLink("Edit", "Edit", new { id = Model.SubscriberId }) |
            @Html.ActionLink("Back to List", "Index")
        </p>
    </div>
</body>
