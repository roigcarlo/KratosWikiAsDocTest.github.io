---
title:  Creating a wrapper of convergence criteria
keywords: 
tags: [Tutorial-Creating-a-wrapper-of-convergence-criteria.md]
sidebar: kratos_sidebar
summary: 
---

# Overview

# Creating a factory
~~~py
from __future__ import print_function, absolute_import, division  # makes KratosMultiphysics backward compatible with python 2.6 and 2.7

# Importing the Kratos Library
import KratosMultiphysics

# Convergence criteria class
class ConvergenceCriteriaFactory(object):
    def __init__(self, convergence_criterion_parameters):
        # Note that all the convergence settings are introduced via a Kratos parameters object.
        
        D_RT = convergence_criterion_parameters["solution_relative_tolerance"].GetDouble()
        D_AT = convergence_criterion_parameters["solution_absolute_tolerance"].GetDouble()
        R_RT = convergence_criterion_parameters["residual_relative_tolerance"].GetDouble()
        R_AT = convergence_criterion_parameters["residual_absolute_tolerance"].GetDouble()
        
        echo_level = convergence_criterion_parameters["echo_level"].GetInt()
        convergence_crit = convergence_criterion_parameters["convergence_criterion"].GetString()
        
        if(echo_level >= 1):
            KratosMultiphysics.Logger.PrintInfo("::[ConvergenceCriterionFactory]:: ", "CONVERGENCE CRITERION : " +
                    convergence_criterion_parameters["convergence_criterion"].GetString())

        if(convergence_crit == "solution_criterion"):
            self.convergence_criterion = KratosMultiphysics.DisplacementCriteria(D_RT, D_AT)
            self.convergence_criterion.SetEchoLevel(echo_level)
            
        elif(convergence_crit == "residual_criterion"):
            self.convergence_criterion = KratosMultiphysics.ResidualCriteria(R_RT, R_AT)
            self.convergence_criterion.SetEchoLevel(echo_level)
                
        elif(convergence_crit == "and_criterion"):
            Displacement = KratosMultiphysics.DisplacementCriteria(D_RT, D_AT)
            Displacement.SetEchoLevel(echo_level)
            Residual = KratosMultiphysics.ResidualCriteria(R_RT, R_AT)
            Residual.SetEchoLevel(echo_level)
            self.convergence_criterion = KratosMultiphysics.AndCriteria(Residual, Displacement)
            
        elif(convergence_crit == "or_criterion"):
            Displacement = KratosMultiphysics.DisplacementCriteria(D_RT, D_AT)
            Displacement.SetEchoLevel(echo_level)
            Residual = KratosMultiphysics.ResidualCriteria(R_RT, R_AT)
            Residual.SetEchoLevel(echo_level)
            self.convergence_criterion = KratosMultiphysics.OrCriteria(Residual, Displacement)
        else:
            err_msg =  "The requested convergence criterion \"" + convergence_crit + "\" is not available!\n"
            err_msg += "Available options are: \"solution_criterion\", \"residual_criterion\", \"and_criterion\", \"or_criterion\""
            raise Exception(err_msg)
~~~

Now if we want to integrate it into the solver, we just need to add the following to the solver:

~~~py
def _create_convergence_criterion(self):
        import base_convergence_criteria_factory as convergence_criteria_factory
        convergence_criterion = convergence_criteria_factory.ConvergenceCriteriaFactory(self._get_convergence_criterion_settings())
        return convergence_criterion.convergence_criterion
~~~
