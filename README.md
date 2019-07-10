During my time at The Tech Academy I worked on the C# Live Project, one of the four projects organized by the Academy. 

There are total 4 Live Projects offered by the Academy to its students: Python Project, C# Project, 
Front End and Back End Projects.

I was working in a team of other several developers on creating an application that was used to manage a collection of 
construction jobs. Project had a list of front-end and back-end user stories. A developer could select any of them to 
work on. Every day our team had a Stand-up meeting there we could discuss what we did yesterday, what we were planning 
to do next, what predicaments we had while coding, and how we could resolve them. 

About the project application:

The application was for a real client of The Tech Academy.
The users of the application were divided into 2 groups: Admins and other Users. Admins were able to create and 
distribute a weekly schedule assigning users to certain jobs. Other Users were able to keep track of which job they are 
assigned to for the week. The application also had the other features such as:  dashboard with company news and 
announcements, and chat functionality for the users to communicate. 
The project was done in ASP.NET MVC 5.

My contributions to the project were several user stories described below:

#### 1. Job Model
Job Model, one of the models of the project, needed to be changed to use a new class ShiftTime that was a list of start times for the week days. 

Job Model:
~~~
namespace ConstructionNew.Models
{
    public class Job
    {
        [Key]
        [Display(Name = "ID")]
        public Guid JobId { get; set; }
        [Display(Name = "Job Title")]
        public string JobTitle { get; set; }
        [Display(Name = "Job#")]
        public string JobNumber { get; set; }
        [Display(Name = "Street Address")]
        public string StreetAddress { get; set; }
        [Display(Name = "City")]
        public string City { get; set; }
        [Display(Name = "State")]
        public State State { get; set; }
        [Display(Name = "Zip")]
        public int? Zipcode { get; set; }
        [Display(Name = "Notes")]
        public string Note { get; set; }
        [Display(Name = "Shift Times")]
        public virtual ShiftTime ShiftTimes { get; set; }

        public virtual ICollection<Schedule> Schedules { get; set; }
    }
}
~~~
After I modified the Job Model, I did migration and populated DB with the following data through the Seed method in the 
Configuration controller:
~~~
var ShiftTimes = new List<ShiftTime>
            {
                new ShiftTime{ShiftTimeId=Guid.Parse("69121893-3afc-4f92-85f3-40bb5e7c7e29"), Default = "08:00"},
                new ShiftTime{ShiftTimeId=Guid.Parse("345df545-678d-cce6-56ab-345df87db234"), Default = "07:30"},
                new ShiftTime{ShiftTimeId=Guid.Parse("2dc45f56-ef98-5634-d743-345abc45dc82"), Default = "09:15"},
            };
            ShiftTimes.ForEach(x => context.ShiftTimes.AddOrUpdate(j => j.ShiftTimeId, x));
            context.SaveChanges();

var Jobs = new List<Job>
            {
                new Job{JobId=Guid.Parse("36c3d745-2db4-4e41-8a39-5cf3ecc84c32"), JobTitle="Hillsboro Aviation",
                        JobNumber="375",StreetAddress ="3845 NE 30th Ave", City = "Hillsboro", State=Enums.State.OR, 
                        Zipcode=97124,Note="Bring hearing protection",
                        ShiftTimes = ShiftTimes.Where(x => x.ShiftTimeId == Guid.Parse("69121893-3afc-4f92-85f3-40bb5e7c7e29")).First()},
   
                new Job{JobId=Guid.Parse("36a8eaa1-7d91-4902-90f4-215827050817"), JobTitle="U-Haul of South Salem", 
                        JobNumber="425", StreetAddress ="1395 12th St SE", City="Salem", State=Enums.State.OR, 
                        Zipcode=97302, Note="",
                        ShiftTimes = ShiftTimes.Where(x => x.ShiftTimeId == Guid.Parse("345df545-678d-cce6-56ab-345df87db234")).First()},
              
               new Job{JobId=Guid.Parse("e7e7a3db-62b9-4eb0-867f-3410c8d28c13"), JobTitle="Olympia  Regional Airport", 
                       JobNumber="475", StreetAddress="7643 Old Hwy 99 SE", City="Olympia", State=Enums.State.WA, 
                       Zipcode=98501, Note="Bring eye protection",
                       ShiftTimes = ShiftTimes.Where(x => x.ShiftTimeId == Guid.Parse("2dc45f56-ef98-5634-d743-345abc45dc82")).First()},
            };
            Jobs.ForEach(x => context.Jobs.AddOrUpdate(j => j.JobId, x));
            context.SaveChanges();
~~~
The existing ShiftTime was set to a new ShiftTime item with Default attribute.

#### 2. Job Create/Edit views 
After migration was done we were switching from a single default start time to a list of start times based on days of 
the week and the default. I needed to modify the Job Create/Edit views to have the Default Value be initial time picker. 
I also added the ability to add another time picker that had a drop down menu for which day of the week it was set for. 
The user could add up to 5 additional pickers for capturing date times.

ShiftTime Model:
~~~
namespace ConstructionNew.Models
{
    public class ShiftTime
    {
        [Key]
        [Display(Name = "Shift Times ID")]
        public Guid ShiftTimeId { get; set; }
        [Display(Name = "Monday")]
        public string Monday { get; set; }
        [Display(Name = "Tuesday")]
        public string Tuesday { get; set; }
        [Display(Name = "Wednesday")]
        public string Wednesday { get; set; }
        [Display(Name = "Thursday")]
        public string Thursday { get; set; }
        [Display(Name = "Friday")]
        public string Friday { get; set; }
        [Display(Name = "Saturday")]
        public string Saturday { get; set; }
        [Display(Name = "Sunday")]
        public string Sunday { get; set; }
        [Display(Name = "Default")]
        public string Default { get; set; }
    }
}
~~~
Added the folowing code to the Job view:
~~~
@Html.Partial("~/Views/Jobs/ModifyShiftTimePartialView.cshtml") <span class="text-danger">@ViewBag.ErrorMessage</span>
~~~
ModifyShiftTimePartialView:
~~~
@{
    ViewBag.Title = "ShiftTimes";
}

@using (Html.BeginForm())
{
    <div class="form-group">
        @Html.Label("Shift Times Default", htmlAttributes: new { @class = "control-label col-md-2" })
        <div class="col-md-2">
            @Html.Editor("ShiftTimeDefault", new { htmlAttributes = new { @class = "form-control timepicker", type = "time", style = "height:40px; width:195px;" } })

        </div>
        <div class="col-md-2">
            <button type="button" class="btn btn-default" name="Modify" id="ShowModify">Modify Shift Times </button>
        </div>

    </div>

    @*Modifyshifttimepartialview only appears when the button  modify shift times  clicked*@

    <div style="display:none;" class="form-group" id="Modify">
        <div class="form-group">
            @Html.Label("      ", htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-2">
                @Html.DropDownList("SelectWeekDay1", ((ConstructionNew.Controllers.JobsController)this.ViewContext.Controller).WeekDays(), "-Select Day-", new { @class = "form-control", type = "day", style = "height:40px; width:195px;" })
            </div>
            <div class="col-md-2">
                @Html.Editor("SelectShiftTime1", new { htmlAttributes = new { @class = "form-control timepicker", type = "time", style = "height:40px; width:195px;" } })

            </div>
        </div>

        <div class="form-group">
            @Html.Label("      ", htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-2">
                @Html.DropDownList("SelectWeekDay2", ((ConstructionNew.Controllers.JobsController)this.ViewContext.Controller).WeekDays(), "-Select Day-", new { @class = "form-control", type = "day", style = "height:40px; width:195px;" })
            </div>
            <div class="col-md-2">
                @Html.Editor("SelectShiftTime2", new { htmlAttributes = new { @class = "form-control timepicker", type = "time", style = "height:40px; width:195px;" } })

            </div>
        </div>

        <div class="form-group">
            @Html.Label("      ", htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-2">
                @Html.DropDownList("SelectWeekDay3", ((ConstructionNew.Controllers.JobsController)this.ViewContext.Controller).WeekDays(), "-Select Day-", new { @class = "form-control", type = "day", style = "height:40px; width:195px;" })
            </div>
            <div class="col-md-2">
                @Html.Editor("SelectShiftTime3", new { htmlAttributes = new { @class = "form-control timepicker", type = "time", style = "height:40px; width:195px;" } })

            </div>
        </div>

        <div class="form-group">
            @Html.Label("      ", htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-2">
                @Html.DropDownList("SelectWeekDay4", ((ConstructionNew.Controllers.JobsController)this.ViewContext.Controller).WeekDays(), "-Select Day-", new { @class = "form-control", type = "day", style = "height:40px; width:195px;" })
            </div>
            <div class="col-md-2">
                @Html.Editor("SelectShiftTime4", new { htmlAttributes = new { @class = "form-control timepicker", type = "time", style = "height:40px; width:195px;" } })

            </div>
        </div>

        <div class="form-group">
            @Html.Label("      ", htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-2">
                @Html.DropDownList("SelectWeekDay5", ((ConstructionNew.Controllers.JobsController)this.ViewContext.Controller).WeekDays(), "-Select Day-", new { @class = "form-control", type = "day", style = "height:40px; width:195px;" })
            </div>
            <div class="col-md-2">
                @Html.Editor("SelectShiftTime5", new { htmlAttributes = new { @class = "form-control timepicker", type = "time", style = "height:40px; width:195px;" } })

            </div>
        </div>
    </div>
}
~~~
Added some methods to the Job Controller and modified the other methods too (please see comments to the code):
~~~
//Converts a list of week days into IEnumerable<SelectListItem> in order to call DropDownList method in Job Create view (created by myself) 
  
        public IEnumerable<SelectListItem> WeekDays()
        {
            List<string> weekDays = new List<string> {"Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday" };
            IEnumerable<SelectListItem> wDays = weekDays.ConvertAll(d =>

                new SelectListItem()
                {
                    Text = d.ToString(),
                    Value = d

                });
            return wDays;
        }
	[HttpPost]
        [ValidateAntiForgeryToken]
        [Authorize(Roles = "Admin")]
        public ActionResult Create([Bind(Include = "JobId,JobTitle,JobNumber,ShiftTimes,StreetAddress,City,State,Zipcode,Note")] Job job, 
                                   [Bind(Include = "Person, StartDate,EndDate")] Schedule schedule, String Foremen,
                                   [Bind(Include = "ShiftTimeId, Default, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday")] ShiftTime shiftTime, string ShiftTimeDefault,
                                   string SelectWeekDay1, string SelectWeekDay2, string SelectWeekDay3, string SelectWeekDay4, string SelectWeekDay5, 
                                   string SelectShiftTime1, string SelectShiftTime2, string SelectShiftTime3, string SelectShiftTime4, string SelectShiftTime5)
        {

            if (ModelState.IsValid)
            {                      
                if (db.Jobs.Any(j => j.JobTitle.Equals(job.JobTitle)))
                {
                    ModelState.AddModelError("JobTitle", "This job title is currently being used, please change the title," +
                                             " or delete/modify the job titled " + "\"" + job.JobTitle + "\".");
                    return View(job);
                }

                if (db.Jobs.Any(j => j.JobNumber.Equals(job.JobNumber)))
                {
                    ModelState.AddModelError("JobNumber", "This job number is currently being used, please change the number," +
                                             " or delete/modify the job numbered " + "\"" + job.JobNumber + "\".");
                    return View(job);
                }

                //Checking if the weekdays repeat (created by myself)

                HashSet<string> selectWD = new HashSet<string>();
                if (!string.IsNullOrEmpty(SelectWeekDay1) && !selectWD.Add(SelectWeekDay1)
                    || !string.IsNullOrEmpty(SelectWeekDay2) && !selectWD.Add(SelectWeekDay2)
                    || !string.IsNullOrEmpty(SelectWeekDay3) && !selectWD.Add(SelectWeekDay3)
                    || !string.IsNullOrEmpty(SelectWeekDay4) && !selectWD.Add(SelectWeekDay4)
                    || !string.IsNullOrEmpty(SelectWeekDay5) && !selectWD.Add(SelectWeekDay5)
                    )
                {
                    ViewBag.ErrorMessage = "Please select any Weekday only once";
                    return View(job);
                }

                else
                {
                    
                    shiftTime.ShiftTimeId = Guid.NewGuid();//Creates ShiftTimeID for a job
                    shiftTime.Default = ShiftTimeDefault;
                 
                    RecordShiftTime(SelectWeekDay1, SelectShiftTime1);
                    RecordShiftTime(SelectWeekDay2, SelectShiftTime2);
                    RecordShiftTime(SelectWeekDay3, SelectShiftTime3);
                    RecordShiftTime(SelectWeekDay4, SelectShiftTime4);
                    RecordShiftTime(SelectWeekDay5, SelectShiftTime5);

                    db.ShiftTimes.Add(shiftTime);
                    db.SaveChanges();
                    job.JobId = Guid.NewGuid();
                    job.ShiftTimes = db.ShiftTimes.Where(j => j.ShiftTimeId == shiftTime.ShiftTimeId).First(); 
                    db.Jobs.Add(job);
                    db.SaveChanges();

                    //Selects the correct collumn in ShiftTimes Table to record a shifttime ( created by myself)
                    ShiftTime RecordShiftTime(string day, string time)
                    {
                       if (day != null)
                        {
                            if (day == "Monday") { shiftTime.Monday = time; }
                            if (day == "Tuesday") { shiftTime.Tuesday = time; }
                            if (day == "Wednesday") { shiftTime.Wednesday = time; }
                            if (day == "Thursday") { shiftTime.Thursday = time; }
                            if (day == "Friday") { shiftTime.Friday = time; }
                            if (day == "Saturday") { shiftTime.Saturday = time; }
                            if (day == "Sunday") { shiftTime.Sunday = time; }
                       }
                        return shiftTime;
                    }

                return RedirectToAction("Index");
                }
            }
            return View(job);
        }

/ POST: Jobs/Edit/5
        // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
        // more details see https://go.microsoft.com/fwlink/?LinkId=317598.

        [HttpPost]
        [ValidateAntiForgeryToken]
        [Authorize(Roles = "Admin")]
        public ActionResult Edit([Bind(Include = "JobId,JobTitle,JobNumber,ShiftTimes,StreetAddress,City,State,Zipcode,Note")] Job job,
                                 [Bind(Include = "ShiftTimeId, Default, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday")] ShiftTime shiftTime)
                             
        {
            if (ModelState.IsValid)
            {   
                //Checking if more than 5 days have been selected ( created by myself)

                List<string> weekDays = new List<string> { job.ShiftTimes.Monday, job.ShiftTimes.Tuesday, job.ShiftTimes.Wednesday,
                                        job.ShiftTimes.Thursday, job.ShiftTimes.Friday, job.ShiftTimes.Saturday, job.ShiftTimes.Sunday};
                List<string> countDays = new List<string>();
                foreach (string d in weekDays)
                {
                    if (d != null)
                    {
                        countDays.Add(d);
                    }
                }
                if (countDays.Count() > 5)
                {
                    ViewBag.ErrorMessage = "Please select no more than 5 Weekdays ";
                    return View(job);
                }
                //Update the DB
                db.Entry(job).State = EntityState.Modified;
                db.Entry(job.ShiftTimes).State = EntityState.Modified;
                db.SaveChanges();
                return RedirectToAction("Index");
            }
            return View(job);
        }
~~~
#### 3.Weekly filter on Schedule Index page
The Schedule Index page had a list of all current schedules. I added a dropdown that had a week time range so the user 
could filter the view for that week only.
Please notice the FetchWeeks method (creates a list of weeks for a year) that I used in my code was written by the other 
team member.

Shedule Index view:
~~~
@using (Html.BeginForm(FormMethod.Post))
{
    <p>
        Filter by week: @Html.DropDownList("SearchWeek", ((ConstructionNew.Controllers.SchedulesController)this.ViewContext.Controller).ListWeeks())
        <input type="submit" value="Search" name="submitButton" />
        <input type="submit" value="All Schedules" name="submitButton" />
    </p>
}
~~~
Schedule Controller:
~~~
	//FetchWeek creates a list of weeks for a year -created by other team member that I used in my code

        public List<string> FetchWeeks(int year)
        {
            List<string> weeks = new List<string>();
            DateTime startDate = new DateTime(year, 1, 1);
            startDate = startDate.AddDays(1 - (int)startDate.DayOfWeek);
            DateTime endDate = startDate.AddDays(6);
            while(startDate.Year < 1 + year)
            {
                weeks.Add(string.Format("{0:MM/dd/yyyy} to {1:MM/dd/yyyy}", startDate, endDate));
                startDate = startDate.AddDays(7);
                endDate = endDate.AddDays(7);
            }
            return weeks;
        }

        //Converts a list of weeks into IEnumerable<SelectListItem> in order to call DropDownList method in 
        //Schedules Index view   

        public IEnumerable<SelectListItem> ListWeeks()
        {
            IEnumerable<SelectListItem> week = FetchWeeks(2019).ConvertAll(w =>
            
                new SelectListItem()
                {
                    Text = w.ToString(),
                    Value = w
                    
                });
            return week;
        }

        //Filters Jobs(Schedules) by Week

        [HttpPost]
        public ViewResult Index(string SearchWeek, string submitButton)
        {     
            if (submitButton == "Search")
            {
                string[] weekParts = SearchWeek.Split(' ');
                DateTime weekStart = Convert.ToDateTime(weekParts[0]);
                DateTime weekEnd = Convert.ToDateTime(weekParts[2]).AddDays(1);

                var jobSearch = db.Jobs.Where(j => j.Schedules.Count > 0).ToList();

                List<Job> result = new List<Job>();
                foreach (Job jobs in jobSearch)
                {
                    foreach (var x in jobs.Schedules)
                    {
                        if ((x.StartDate >= weekStart) && (x.StartDate <= weekEnd)
                           || (x.EndDate <= weekEnd) && (x.EndDate >= weekStart)
                           || (x.StartDate <= weekStart) && (x.EndDate >= weekEnd)
                           || (x.StartDate <= weekStart) && (x.EndDate == null))
                           {
                            result.Add(jobs);
                           }
                    }                   
                }          
                return View(result);
            } else
            {
                var jobs = db.Jobs.Where(j => j.Schedules.Count > 0).ToList();
                return View(jobs);
            }                                 
        }
~~~
#### 4. Date picker in the Company News Edit view
I added a date picker for expiration date and pre-set it to the existing date of the item in the Company News Edit view:

Company News Edit view:
~~~
<div class="form-group">
         @Html.LabelFor(model => model.ExpirationDate, htmlAttributes: new { @class = "control-label col-md-2" })
         <div class="col-md-10">
             @Html.EditorFor(model => model.ExpirationDate, new { htmlattributes = new { @class = "datepicker form-control", style = "width:200px;" } })
             @Html.ValidationMessageFor(model => model.ExpirationDate, "", new { @class = "text-danger" })
          </div>
</div>

@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
    <script type="text/javascript">
        $(function () { // will trigger when the document is ready
            $('.datepicker').datepicker(); //Initialise any date pickers
        });
    </script>
}
~~~
Company News Model:
~~~
namespace ConstructionNew.Models
{
    public class CompanyNews
    {
        [Key]
        [Display(Name = "Date")]
        [DisplayFormat(DataFormatString = "{0:MM/dd/yy hh:mm tt}")]
        public string DateStamp { get; set; }
        [Display(Name = "Title")]
        public string Title { get; set; }
        [Display(Name = "Body")]
        public string NewsItem { get; set; }
        [DataType(DataType.Date)]
        [Display(Name = "Expiration Date")]
        [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
        public Nullable<DateTime> ExpirationDate { get; set; }
    }
}
