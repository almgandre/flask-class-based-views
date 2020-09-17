from flask import (
    render_template,
    redirect,
    request,
    url_for,
    flash,
    abort
)

from flask.views import View


class TemplateView(View):
    methods = ['GET']

    def get_template_name(self):
        if not self.template_name:
            raise NotImplementedError()
        return str(self.template_name)

    def get_context_data(self, **kwargs):
        return kwargs

    def get(self, **kwargs):
        pass

    def render_template(self, **kwargs):
        kwargs.update(self.get_context_data(**kwargs))
        return render_template(self.get_template_name(), **kwargs)

    def dispatch_request(self, **kwargs):

        if request.method == 'GET':
            self.get(**kwargs)

        return self.render_template(**kwargs)


class ListView(TemplateView):

    def get_objects(self):
        return self.model.query.all()

    def get_context_data(self, **kwargs):
        context = super(ListView, self).get_context_data(**kwargs)
        context['object_list'] = self.get_objects()
        return context


class SingleObjectMixin(object):

    def get_object(self, pk):
        obj = self.model.query.filter_by(id=pk).first()
        if not obj:
            abort(404)
        return obj

    def form_valid(self, form):

        if not self.object:
            self.object = self.model()

        form.populate_obj(self.object)
        self.object.save()

        return super(SingleObjectMixin, self).form_valid(form)


class FormView(TemplateView):
    methods = ['GET', 'POST']
    success_msg = None

    def get_success_url(self):
        if not self.success_url:
            raise NotImplementedError()
        return url_for(str(self.success_url))

    def get_form(self, obj=None):
        if not self.form_class:
            raise NotImplementedError()
        if obj:
            return self.form_class(obj=obj)
        return self.form_class()

    def dispatch_request(self, **kwargs):
        if kwargs.get('pk') or kwargs.get('id'):
            pk = kwargs.get('pk') or kwargs.get('id')
            self.object = self.get_object(pk)
            form = self.get_form(obj=self.object)
        else:
            self.object = None
            form = self.get_form()

        if request.method == 'POST':
            return self.post(form)

        return super(FormView, self).dispatch_request(form=form, **kwargs)

    def post(self, form):
        if form.validate_on_submit():
            return self.form_valid(form)
        else:
            return self.form_invalid(form)

    def form_invalid(self, form):
        return self.render_template(form=form)

    def form_valid(self, form):
        if self.success_msg:
            flash(self.success_msg, 'success')

        return redirect(self.get_success_url())


class CreateView(SingleObjectMixin, FormView):
    success_msg = "Record created successfully!"


class UpdateView(SingleObjectMixin, FormView):
    success_msg = "Record edited successfully!"

    def get_context_data(self, **kwargs):
        context = super(UpdateView, self).get_context_data(**kwargs)
        context['object'] = self.object
        return context


class DeleteView(SingleObjectMixin, TemplateView):
    success_msg = "Record deleted successfully!"

    def get_success_url(self):
        if not self.success_url:
            raise NotImplementedError()
        return url_for(str(self.success_url))

    def delete(self):
        self.object.delete()

    def dispatch_request(self, pk):
        self.object = self.get_object(pk)
        self.delete()

        flash(self.success_msg, 'success')
        return redirect(self.get_success_url())


class RedirectView(View):
    methods = ['GET']

    def get_redirect_url(self):
        if not self.redirect_url:
            raise NotImplementedError()
        return url_for(str(self.redirect_url))

    def dispatch_request(self):
        return redirect(self.get_redirect_url())

