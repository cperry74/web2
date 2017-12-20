web service workflow


example: https://XXX.XXX.com/content/AppName/Config_Name/assignments/

web2.urls.py:
url(r'^AppName/(?P<configName>\w+)/', include('web2.AppName.urls')),

web2.AppName.urls.py:
url(r'^assignments/$', views.AssignmentsList),

web2.AppName.views.py.AssignmentsList:

def WithConfig(method):
    def wrapped(request, configName, assignmentId = None, **kwargs):
        config = _ConfigForName(configName)
        if not config.canViewAssignments: 
            return access_not_configured(request)
        if assignmentId is None:
            return method(request, config, **kwargs)
        assignment = config.cls.GetObjectOr404(assignmentId = assignmentId)
        return method(request, config, assignment = assignment, **kwargs)
    return wrapped

@WithConfig
def AssignmentsList(request, config):
    assignments = config.cls.GetObjects()
    historicalAssignments = [a for a in assignments if a.historical]
    activeAssignments = [a for a in assignments if a.active]
    requestedAssignments = [a for a in assignments if a.requested]
    myAssignments = [a for a in assignments if a.isLead]
    handler = config.GetHandler(request.context)
    reports = handler.GetReports()
    return render_to_response(request, "AppName/AssignmentList.html",
            config = config, activeAssignments = activeAssignments,
            historicalAssignments = historicalAssignments,
            requestedAssignments = requestedAssignments,
            myAssignments = myAssignments, menuItems = handler.GetMenuItems(),
            attributes = config.cls.GetAttributesForUser(), reports = reports)

def _ConfigForName(configName):
    config = models.Configs.GetObjectOr404(name = configName)
    callproc("pkg_Common.SetConfiguration", config.configId)
    config.cls = config.GetCustomModel("Assignments")
    config.overtimeRequestModel = config.GetCustomModel("OvertimeRequests")
    return config

web2.pkg_common.SetConfiguration()
  procedure SetConfiguration (
      a_ConfigId                      number
  ) is
  begin
      common.pkg_Context.SetConfiguration(a_ConfigId);
  end;

common.pkg_Context.SetConfiguration()
    procedure SetConfiguration (
        a_ConfigId                      number
    ) is
    begin
        dbms_session.set_context('edscontext', 'ConfigId', a_ConfigId);
    end;

templates.AppName.AssignmentList.html:
