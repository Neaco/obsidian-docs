
The main risk is that teams can accidentally claim hosts/paths they don't own, causing route collisions or traffic hijacking across tenants through the shared ingress controller.


Solution : enforce a namespace-scoped hostname pattern (\<app>.\<namespace>.example.com) so teams can only create Ingresses for domains they own